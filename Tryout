import openai
import json
import time
from tqdm import tqdm

output_file = "output.md"

# Spielparameter

total_rounds = 7
payoff_matrix = {
    ('c', 'c'): (7, 7),
    ('c', 'd'): (0, 15),
    ('d', 'c'): (15, 0),
    ('d', 'd'): (2, 2)
}

# features
features_self ="think strategically and try to maximize your payouts. Behave authentically and true to your own values, without being explicitly fair or selfish."
features_fair = "think strategically and try to maximize his payouts. Behave fair."
features_selfish = "think strategically and try to maximize his payouts. Behave selfish."

short_feature1=""
short_feature2=""


history_player1 = []
history_player2 = []

player_1_earnings = 0
player_2_earnings = 0

def build_prompt(round_number, player_number, your_features, opponent_features, round_history_str, your_earnings, opponent_earnings):
    if round_number == 1:
        prompt = f'''
Game Explanation:
In this game you are Player {player_number} and will play an unknown number of rounds of a game with another player.
Both of you simultaneously choose either option c or d which will result in different outcomes:
1. if both of you choose c, both of you get 7€
2. if one of you chooses c and the other d, the c player gets 0€ and the d player gets 15€
3. if both of you choose d, both of you get 2€

read the outcomes, remember them and choose carefully in each round while trying to maximize your payoff.
From round 2 onward you will be given a game history with the choices and payoffs of you and the other player.

your instruction: {your_features}
potential opponent instruction: {opponent_features}

Lets start with round 1. 
remember your instruction and the other players instruction carefully, also remember that you are player {player_number}

Please provide your answer in this round in a single-line JSON format with two keys:  “reasoning” and “decision”  
{{"reasoning": "...", "decision": "..."}}
at the beginning of the "reasoning" key, state your player number and briefly explain your reasoning before making the decision.
The value of “decision” should be either “c” or “d”.  
Important: {your_features} and your opponent might {opponent_features}
'''
    else:
        history_str="\n".join(history_player1 if player_number == 1 else history_player2)
        prompt = f'''
    Recall that you are the player {player_number}. 
You have played {round_number} round(s) before. 
Here is the history of the gameplay of previous rounds: 
{history_str}

Your total payoff so far: {your_earnings} euros;  
The other player's total payoff so far: {opponent_earnings} euros.

Please provide your answer in this round in a single-line JSON format with two keys:  “reasoning” and “decision”  
{{"reasoning": "...", "decision": "..."}}
at the beginning of the "reasoning" key, state your player number and briefly explain your reasoning before making the decision.
The value of “decision” should be either “c” or “d”.
This is round {round_number} of the game with unknown rounds left.  
Important: {your_features} and your opponent might {opponent_features}.
'''
    return prompt.strip()


def ask_gpt(system_message, prompt):
    client=openai.OpenAI()
    response = openai.chat.completions.create(
        model="gpt-4.1",
        messages=[
            {"role": "system", "content": system_message},
            {"role": "user", "content": prompt}
        ],
        temperature=0.7,
    )
    return response.choices[0].message.content

for setups_number in tqdm(range(1,4),desc="setups"):
    if setups_number == 1:
        features_opponent1=features_fair
        features_opponent2 = features_selfish
        short_feature1 = "fair"
        short_feature2 = "selfish"
    elif setups_number == 2:
        features_opponent1 = features_opponent2 = features_selfish
        short_feature1 = "selfish"
        short_feature2 = "selfish"
    else:
        features_opponent1 =features_opponent2= features_fair
        short_feature1 = "fair"
        short_feature2 = "fair"
    for game_number in tqdm(range(1,11), desc="games", leave=False):
        history_player1 = []
        history_player2 = []
        player_1_earnings = 0
        player_2_earnings = 0
        with open(output_file, "a", encoding="utf-8") as f:
            f.write(f'''\n## Setup: {short_feature1} vs {short_feature2}\n### Game: {game_number}\n''')
            for round_num in tqdm(range(1, total_rounds + 1), desc=f"Round {game_number}", leave=False):
                # Player 1 Prompt
                system_1 = f"You will play a multi round game. Think before you make a decision and {features_self}"
                prompt_1 = build_prompt(round_num, 1, features_self, features_opponent1, history_player1, player_1_earnings, player_2_earnings)
                output_1 = ask_gpt(system_1, prompt_1)
                f.write(f"\n#### Player 1 output round {round_num}:\n{output_1}")

                # Player 2 Prompt
                system_2 = f"You will play a multi round game. Think before you make a decision and {features_self}"
                prompt_2 = build_prompt(round_num, 2, features_self, features_opponent2, history_player2, player_2_earnings, player_1_earnings)
                output_2 = ask_gpt(system_2, prompt_2)
                f.write(f"\n#### Player 2 output round {round_num}:\n{output_2}")

                # Extrahiere Entscheidung
                try:
                    decision_1 = json.loads(output_1)["decision"].strip().lower()
                    decision_2 = json.loads(output_2)["decision"].strip().lower()
                except:
                    print("⚠️ Fehler beim Parsen der Antwort – überprüfe das JSON-Format.")
                    break

                # Berechne Auszahlung
                p1, p2 = payoff_matrix[(decision_1, decision_2)]
                player_1_earnings += p1
                player_2_earnings += p2


                # For player 1's perspective:
                round_summary_1 = (
                    f"Round {round_num} summary: you chose {decision_1} and the other player chose {decision_2}. "
                    f"This results in you getting {p1} and the other player getting {p2}. "
                    f"Your total payoff is {player_1_earnings} and the other player's is {player_2_earnings}"
                )
                # For player 2's perspective:
                round_summary_2 = (
                    f"Round {round_num} summary: you chose {decision_2} and the other player chose {decision_1}. "
                    f"This results in you getting {p2} and the other player getting {p1}. "
                    f"Your total payoff is {player_2_earnings} and the other player's is {player_1_earnings}"
                )

            # Store both histories
                history_player1.append(round_summary_1)
                history_player2.append(round_summary_2)


                time.sleep(2)  # Rate limit beachten
            #Spiel beendet
            f.write("\n#### SPIEL BEENDET")
            f.write(f"\n**Player 1 total payoff: {player_1_earnings}€**")
            f.write(f"\n**Player 2 total payoff: {player_2_earnings}€**\n")
            
            # Ask both players about their belief regarding the opponent's instruction
            def belief_prompt(player_number):
                history = "\n".join(history_player1 if player_number == 1 else history_player2)
                prompt = (
                    "That was the last round of the game. Here is the full history of choices and payoffs:\n"
                    f"{history}\n"
                    "In this game, 'c' is considered the fair/cooperative choice, and 'd' is considered the selfish/defective choice.\n"
                    "Based on this, do you believe the instruction your opponent was given was true or not? "
                    "Please answer in a single-line JSON: "
                    '{"reasoning": "...", "belief": "true" or "false"}'
                )
                return prompt
            
            # Player 1's belief about Player 2's instruction
            system_1_belief = f"You are Player 1. {features_self}"
            prompt_1_belief = (
                f"{belief_prompt(1)}\n"
                f"Your instruction: {features_self}\n"
                f"potential opponent's instruction: {features_opponent1}"
            )
            output_1_belief = ask_gpt(system_1_belief, prompt_1_belief)
            f.write(f"\n#### Player 1 belief about Player 2's instruction:\n{output_1_belief}")
            
            # Player 2's belief about Player 1's instruction
            system_2_belief = f"You are Player 2. {features_self}"
            prompt_2_belief = (
                f"{belief_prompt(2)}\n"
                f"Your instruction: {features_self}\n"
                f"potential opponent's instruction: {features_opponent2}"
            )
            output_2_belief = ask_gpt(system_2_belief, prompt_2_belief)
            f.write(f"\n#### Player 2 belief about Player 1's instruction:\n{output_2_belief}\n")