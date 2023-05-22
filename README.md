# collabHAI_pref Documentation

## Information About game_logs.db (Game Logs)
*Last updated: 5/22/2023

After a game is completed, the JS client sends data to the Python server (via a Websocket) to log game informaiton in an [SQLite](https://www.sqlite.org/index.html) database. Once the green text at the botom of the "game ended" screen changes from "Loading..." to a valid completion code, the data has been successfully saved to the database.

The game_logs.db consists of 6 tables:
1. [Games](#1-games)
2. [Frames](#2-frames)
3. [Actions](#3-actions)
4. [Bullets](#4-bullets)
5. [Enemies](#5-enemies)
6. [Events](#6-events)


### 1. Games
Each row in the Games table represents one completed round of the game. 

The Games table has 4 columns:
- **id**: the row number; **game_id** for game used to identify the game in [Frames](#2-frames) table (integer type)
- **player_id**: the ID passed in via the `id=` URL parameter (default is `UNDEFINED`, text type)
- **date**: timestamp of when game was created (text type)
- **mode**: the mode of the game passed in via the `mode=` URL parameter (default is 1, integer type)
    - 1 (*default*)= Cooperative Early: agent comes over to help twice before all of own enemies destroyed
    - 2 = Cooperative Late: agent comes over to help after all of own enemies destroyed
    - 3 = Uncooperative: agent stays on right side


### 2. Frames
Each row of the Frames table represents one frame of a game. A frame is one pass through the `update()` function for the game.

The Frames tables has 11 columns:
- **id**: the row number; **frame_id** used in other tables to identigy the corresponding frame (don't use true **frame_number** in other tables because you would need a double lookup with **game_id**, integer type)
- **game_id**: matches **id** from [Games](#1-games) table (integer type)
- **timestamp**: timestamp for each frame (text type)
- **frame_number**: frame_number from game loop (integer type)
- **player_position**: x-coordinate for player ship (integer type)
- **player_lives**: number of lives remaining for player (integer type)
- **player_score**: total points scored up to this frame (integer type)
- **ai_position**: x-coordinate for agent ship (integer type)
- **ai_lives**: number of lives remaining for the agent ship (integer type)
- **ai_score**: total points scored up to this frame (integer type)
- **frame_sent**: indicator variable if frame was sent via control socket (integer type)
    - O: not sent
    - 1: sent


### 3. Actions
Each row of the Actions table represents the player and AI actions from one frame of a game. A frame is one pass through the `update()` function for the game.

The Actions tables has 17 columns:
- **game_id**: matches **id** from [Games](#1-games) table (integer type)
- **frame_id**: matches **id** from [Frames](#2-frames) table (integer type)
- **frame_sent**: indicator variable if frame was sent via control socket (integer type)
    - O: not sent
    - 1: sent
- **player_left**: indicator variable if player tried to move left (if **player_left** = 1 AND **frame_sent** = 1, player did move left, integer type)
    - 0: player did not press left arrow to move left
    - 1: player did press left arrow to move left
- **player_right**: indicator variable if player tried to move right (if **player_right** = 1 AND **frame_sent** = 1, player did move right, integer type)
    - 0: player did not press right arrow to move right
    - 1: player did press right arrow to move right
- **player_shoot**: indicator variable if player tried to shoot and game rules would allow (integer type)
    - 0: player did not press space bar to shoot or game rules did not allow player to shoot (e.g., not enough time since last shot)
    - 1: player did press space bar to shoot and game rules allowed it
- **player_tried**: indicator variable if player tried to shoot (integer type)
    - 0: player did not press space bar to shoot
    - 1: player did press space bar to shoot
- **player_signal_down**: indicator variable if player tried giving negative feedback with down arrow key and game rules would allow (integer type)
    - 0: player did not press down arrow key to give negative feedback or game rules did not allow
    - 1: player did press down arrow key to give negative feedback and game rules allowed it
- **player_signal_up**: indicator variable if player tried giving positive feedback with up arrow key and game rules would allow (integer type)
    - 0: player did not press up arrow key to give positive feedback or game rules did not allow
    - 1: player did press up arrow key to give positive feedback and game rules allowed it
- **player_tried_down**: indicator variable if player tried giving negative feedback with down arrow key (integer type)
    - 0: player did not press down arrow key to give negative feedback
    - 1: player did press down arrow key to give negative feedback
- **player_tried_up**: indicator variable if player tried giving positive feedback with up arrow key (integer type)
    - 0: player did not press up arrow key to give positive feedback
    - 1: player did press up arrow key to give positive feedback 
- **ai_actual_left**: (integer type)
- **ai_actual_right**: (integer type)
- **ai_actual_shoot**: (integer type)
- **ai_rec_left**: (integer type)
- **ai_rec_right**: (integer type)
- **ai_rec_shoot**: (integer type)


### 4. Bullets
Each row in the Bullets table represents the positioning and of a bullet at a specific frame of one game. A frame is one pass through the `update()` function for the game.

The Bullets table has 5 columns:
- **id**: the row number, used to identify the specific bullet (integer type)
- **frame_id**: matches **id** from [Frames](#2-frames) table (integer type)
- **type**:  (integer type)
- **x**: x position of the bullet at the current frame (integer type)
- **y**: y position of the bullet at the current frame (integer type)


### 5. Enemies
Each row in the Enemies table represents the positioning and of an enemy at a specific frame of one game. A frame is one pass through the `update()` function for the game.

The Enemies table has 5 columns:
- **id**: the row number, used to identify the specific enemy (integer type)
- **frame_id**: matches **id** from [Frames](#2-frames) table (integer type)
- **side**: specifices the side of the screen (player or AI) the enemy is in at the current frame (text type)
- **x**: x position of the enemy at the current frame (integer type)
- **y**: y position of the enemy at the current frame (integer type)


### 6. Events
Each row in the Events table represents a frame where the player or the AI kills an enemy. A frame is one pass through the `update()` function for the game.

The Events table has 4 columns:
- **id**: the row number, used to identify the specific event incident (integer type)
- **frame_id**: matches **id** from [Frames](#2-frames) table, the specific frame at which the killing occurred (integer type)
- **killer**: represents who killed the enemy (text type)
    - PLAYER: the player killed the enemy at the specific frame
    - AI: the AI agent killed the enemy at the specific frame
- **killed**: represents which side the enemy was killed in (text type)
    - LEFT: the enemy was killed on the left side of the screen at the specific frame
    - RIGHT: the enemy was killed on the right side of the screen at the specific frame


## Information About HRI2023Analysis.ipynb (Analysis Dataframes)
*Last updated: 5/22/2023

Based on the Qualtrics survey and some of the game and control logs, the HRI2023Analysis.ipynb Jupyter notebook is used to process the data from the game logs and from the survey. The results are exported into a CSV file that can be used for data analysis where each row is a different participant.

### Dataframe Variables
The following are variables used in the CSV and data analysis that are originally based on the Qualtrics CSV variables.

- **StartDate**: the exact date of the interaction start time in the format yyyy-mm-dd hh:mm:ss (24 hour system)
- **EndDate**: the exact date of the interaction end time in the format yyyy-mm-dd hh:mm:ss (24 hour system)
- **Status**: 
- **IPAddress**: the IP Address of the device used for the Qualtrics survey (all 130.132.173.36 since the study used one device for all participants to standardize)
- **Progress**: the participant's progress on the Qualtrics survey (scale of 0-100)
- **Duration (in seconds)**: the amount of time in seconds that the participant took to complete the Qualtrics survey
- **Finished**: represents if the participant completed the entire Qualtrics survey
    - O: Qualtrics survey is not finished or incomplete
    - 1: Qualtrics survey is finished or complete
- **RecordedDate**: the exact date of the interaction recorded time in the format yyyy-mm-dd hh:mm:ss (24 hour system)
- **ResponseId**: 
