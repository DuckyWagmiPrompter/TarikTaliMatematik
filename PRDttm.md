# PRD — Tarik Tali Matematik
Version: 1.0  
Product Type: Educational Web Game  
Theme: Heritage Reimagined (Malaysia Traditional Game Digitalization)

---

# 1. Product Overview

## Product Name
Tarik Tali Matematik

## Vision
Reimagine Malaysia’s traditional “Tarik Tali” (Tug of War) into an interactive digital learning experience that teaches mathematics while preserving cultural play values.

## Mission
Encourage children to learn mathematics through competitive gameplay, motivation mechanics, and Malaysian cultural elements.

## Target Users
- Primary school children (Age 6–12)
- Parents seeking educational games
- Schools
- Casual learners
- Multiplayer classroom environment

---

# 2. Problem Statement

### Current Problems
- Kids disengage from traditional math learning
- Traditional Malaysian games are fading
- Educational apps lack fun competition mechanics
- Screen time often lacks learning value

### Solution
Gamify math using tug-of-war mechanics to:
- make learning addictive
- build cognitive reflex
- preserve heritage gameplay
- introduce cooperative learning

---

# 3. Product Goals

### Business Goals
- Preserve Malaysian heritage digitally
- Promote gamified education
- Build scalable multiplayer learning platform
- Create engaging STEM learning product

### User Goals
- Learn math while playing
- Feel rewarded through gameplay
- Compete with friends
- Improve mental speed

---

# 4. Core Game Concept

## Gameplay Loop

1. Player receives math question
2. Player selects answer
3. Correct answer → rope pulled
4. Wrong answer → opponent pulls
5. First to reach threshold wins

## Educational Layer
- Increasing difficulty
- Adaptive progression
- Quick mental calculation training

## Cultural Layer
- Forest environment
- Malaysian language prompts
- Traditional tug-of-war concept

---

# 5. Features

## 5.1 Core Features (MVP)

### Single Player Mode
- AI opponent
- Progressive difficulty
- Score tracking
- Win/Lose system

### Multiplayer Mode
- Real-time room system
- Firebase sync
- Turn-based answering
- Shareable invite link

### Math Engine
- Dynamic question generation
- Addition
- Subtraction
- Multiplication (higher difficulty)

### Tug-of-War Mechanics
- Rope position physics
- Power pull system
- Win threshold
- Lose threshold

### Feedback System
- Correct / Wrong animation
- Streak bonus
- Motivation messages

### Motivational Companion
- Animal coach
- Encouragement quotes
- Positive reinforcement

---

## 5.2 Engagement Features

- Streak multiplier
- Difficulty scaling
- Avatar system
- Leaderboard UI
- Score system
- Visual progress bar

---

## 5.3 Social Features

- Room-based multiplayer
- Invite link sharing
- Turn indicator
- Player avatars

---

# 6. Technical Architecture

## Frontend
- React
- Tailwind CSS
- SVG vector characters
- State-driven UI

## Backend
- Firebase Authentication
- Firebase Firestore realtime sync
- Anonymous login
- Room-based multiplayer state

## Data Model

### Room Object
room {
host
guest
status
currentTurn
score
}


### Question Model
question {
text
answer
options
difficulty
}


---

# 7. Game Mechanics

## Difficulty Scaling
difficulty = floor(score / 500) + 1


## Power Pull
power = 8 + streak multiplier


## Win Condition
rope >= 90 → win
rope <= 10 → lose


---

# 8. Non Functional Requirements

- Fast UI response
- Realtime sync < 200ms
- Offline fallback mode
- Child friendly UX
- Mobile responsive
- Secure multiplayer sessions

---

# 9. Success Metrics

- Session duration
- Questions answered per session
- Multiplayer usage rate
- Learning accuracy rate
- Repeat usage

---

# 10. Future Enhancements (Post MVP)

- Malaysian traditional themes (Congkak mode, Gasing mode)
- Teacher dashboard
- Classroom leaderboard
- AI adaptive learning
- Voice input answers
- Bahasa Melayu / English switch
- Analytics dashboard

---