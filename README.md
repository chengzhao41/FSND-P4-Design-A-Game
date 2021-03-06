#Full Stack Nanodegree Project 4 Design a Game

## Set-Up Instructions:
1.  Update the value of application in app.yaml to the app ID you have registered
 in the App Engine admin console and would like to use to host your instance of this sample.
1.  Run the app with the devserver using dev_appserver.py DIR, and ensure it's
 running by visiting the API Explorer - by default localhost:8080/_ah/api/explorer.
1.  (Optional) Generate your client library(ies) with the endpoints tool.
 Deploy your application.
 
 
 
##Game Description:
Tic-tac-toe (also known as Noughts and crosses or Xs and Os) is a game for two players, X and O, who take turns marking the spaces in a 3×3 grid. The player who succeeds in placing three of their marks in a horizontal, vertical, or diagonal row wins the game.
Many different games can be played by many different Users at any given time. Each game can be retrieved or played by using the path parameter `urlsafe_game_key`.

##Score Keeping:
The score will be based on the highest net win to game ratio, which will be defined as (# of wins - # of losses) / (# of games).


##Files Included:
 - api.py: Contains endpoints and game playing logic.
 - app.yaml: App configuration.
 - cron.yaml: Cronjob configuration.
 - main.py: Handler for taskqueue handler.
 - models.py: Entity and message definitions including helper methods.
 - utils.py: Helper function for retrieving ndb.Models by urlsafe Key string.

##Endpoints Included:
 - **create_user**
    - Path: 'user'
    - Method: POST
    - Parameters: user_name, email (optional), email_remainder (optional)
    - Returns: Message confirming creation of the User.
    - Description: Creates a new User. user_name provided must be unique. Will 
    raise a ConflictException if a User with that user_name already exists.

  - **set_email_preference**
    - Path: 'user/{urlsafe_user_key}'
    - Method: POST
    - Parameters: urlsafe_user_key, email (optional), email_remainder (optional)
    - Returns: Message confirming current email preference.
    - Description: Modifies the email and email remainder field for a current user. Will 
    raise a ConflictException if a User with that urlsafe_user_key does not exist.

  - **get_email_preference**
    - Path: 'user/{urlsafe_user_key}'
    - Method: GET
    - Parameters: user_name
    - Returns: Message confirming current email preference.
    - Description: Gets the email and email remainder field for a current user. Will 
    raise a ConflictException if a User with that urlsafe_user_key does not exist.
    
 - **new_game**
    - Path: 'game'
    - Method: POST
    - Parameters: user_name1, user_name2
    - Returns: GameForm with initial game state.
    - Description: Creates a new Game. user_name1 and user_name2 provided must correspond to an
    existing user - will raise a NotFoundException if not. 
     
 - **get_game**
    - Path: 'game/{urlsafe_game_key}'
    - Method: GET
    - Parameters: urlsafe_game_key
    - Returns: GameForm with current game state.
    - Description: Returns the current state of a game.
    
 - **make_move**
    - Path: 'game/{urlsafe_game_key}'
    - Method: PUT
    - Parameters: urlsafe_game_key, board_position
    - Returns: GameForm with new game state.
    - Description: Accepts a move in the form of a comma seperated string, with each 'X', 'O', or ' ' indicating the new board position from the top left corner to the buttom right corner and returns the updated state of the game.
    If this causes a game to end, a corresponding Score entity will be created.
    
 - **get_scores**
    - Path: 'scores'
    - Method: GET
    - Parameters: None
    - Returns: ScoreForms.
    - Description: Returns all Scores in the database (unordered).
    
 - **get_user_scores**
    - Path: 'scores/user/{user_name}'
    - Method: GET
    - Parameters: user_name
    - Returns: ScoreForms. 
    - Description: Returns all Scores recorded by the provided player (unordered).
    Will raise a NotFoundException if the User does not exist.
    
 - **get_active_game_count**
    - Path: 'games/average_moves_per_game'
    - Method: GET
    - Parameters: None
    - Returns: StringMessage
    - Description: Gets the average number of moves madefor all finished games from a previously cached memcache key.

##Models Included:
 - **User**
    - Stores unique user_name, (optional) email address, (optional) email remainder.
    
 - **Game**
    - Stores unique game states. Associated with User model via KeyProperty.
    
 - **Score**
    - Records completed games. Associated with Users model via KeyProperty.
    
##Forms Included:
 - **GameForm**
    - Representation of a Game's state (urlsafe_key, board_position as a comma seperated string with each 'X', 'O', or ' ' representing the piece on a 3x3 board from the top left going to the bottom right from left to right and top to bottom,
    game_over flag, message, user_name1, user_name2).
 - **NewGameForm**
    - Used to create a new game (user_name1, user_name2)
 - **MakeMoveForm**
    - Inbound make move form (board_position).
 - **ScoreForm**
    - Representation of a completed game's Score (user_name, date, won flag, lost flag).
 - **ScoreForms**
    - Multiple ScoreForm container.
 - **StringMessage**
    - General purpose String container.    

## Design decisions:
The design was based on the provided sample game of Guess-a-Number. The main challenge was how to encode the board position, and I decided to use a string to do so in the form 9 comma-seperated characters, where each character has to be a ' ', 'X', or 'O'. This make it easy to store in the datastore, but the down-side is that we need to split the string before we can apply logic on the board position. 