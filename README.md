# RockPaperScissorsAI

This project is a fun way to play Rock-Paper-Scissors using a webcam. It detects your hand gestures using computer vision and Mediapipe, recognizing whether you're showing "Rock," "Paper," or "Scissors." The program compares your move with a randomly chosen move from the computer. Based on the classic game rules, it determines the winner. A countdown timer ensures fair play, giving you time to show your gesture before the match starts. The result is displayed on the screen along with both moves. It's an interactive blend of artificial intelligence and hand tracking, turning a simple game into an engaging digital experience.

code:


 import cv2
import mediapipe as mp
import random
import time

mp_hands = mp.solutions.hands
mp_drawing = mp.solutions.drawing_utils


# Helper: Count raised fingers
def count_fingers(hand_landmarks):
    tips = [4, 8, 12, 16, 20]  # Thumb, Index, Middle, Ring, Pinky
    fingers = []

    # Thumb (compare x not y)
    if hand_landmarks.landmark[tips[0]].x < hand_landmarks.landmark[tips[0] - 1].x:
        fingers.append(1)
    else:
        fingers.append(0)

    # Rest fingers
    for tip in tips[1:]:
        if hand_landmarks.landmark[tip].y < hand_landmarks.landmark[tip - 2].y:
            fingers.append(1)
        else:
            fingers.append(0)

    return sum(fingers)


# Map finger count to RPS gesture
def get_gesture(finger_count):
    if finger_count == 0:
        return "Rock"
    elif finger_count == 2:
        return "Scissors"
    elif finger_count == 5:
        return "Paper"
    else:
        return "Unknown"


# Decide winner
def get_winner(user, computer):
    if user == computer:
        return "Draw"
    if (user == "Rock" and computer == "Scissors") or \
            (user == "Scissors" and computer == "Paper") or \
            (user == "Paper" and computer == "Rock"):
        return "You Win!"
    return "Computer Wins!"


# Initialize webcam
cap = cv2.VideoCapture(0)

with mp_hands.Hands(max_num_hands=1, min_detection_confidence=0.7) as hands:
    last_time = time.time()
    countdown = 5
    computer_move = None
    result = ""

    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break

        frame = cv2.flip(frame, 1)
        h, w, _ = frame.shape
        rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        result_hands = hands.process(rgb)

        user_move = "Waiting..."

        if result_hands.multi_hand_landmarks:
            for hand_landmarks in result_hands.multi_hand_landmarks:
                mp_drawing.draw_landmarks(frame, hand_landmarks, mp_hands.HAND_CONNECTIONS)
                fingers_up = count_fingers(hand_landmarks)
                user_move = get_gesture(fingers_up)

        # Timer logic
        elapsed = time.time() - last_time
        if elapsed > countdown:
            if user_move != "Unknown":
                computer_move = random.choice(["Rock", "Paper", "Scissors"])
                result = get_winner(user_move, computer_move)
                last_time = time.time()  # restart timer
        else:
            cv2.putText(frame, f"Show Gesture in {int(countdown - elapsed)}", (10, 40),
                        cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 0, 255), 2)

        # Display moves
        cv2.putText(frame, f"Your Move: {user_move}", (10, 100),
                    cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 0, 0), 2)
        if computer_move:
            cv2.putText(frame, f"Computer: {computer_move}", (10, 150),
                        cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 0, 0), 2)

        # Result
        if result:
            cv2.putText(frame, f"Result: {result}", (10, 200),
                        cv2.FONT_HERSHEY_SIMPLEX, 1.2, (0, 255, 0), 3)

        cv2.imshow("Rock Paper Scissors", frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

cap.release()
cv2.destroyAllWindows()
