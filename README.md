 import pygame
 import sys

 # Initialize pygame
 pygame.init()

 # Constants
 WIDTH, HEIGHT = 600, 700  # Extra height for status display
 LINE_WIDTH = 15
 BOARD_SIZE = 3
 SQUARE_SIZE = WIDTH // BOARD_SIZE
 CIRCLE_RADIUS = SQUARE_SIZE // 3
 CIRCLE_WIDTH = 15
 CROSS_WIDTH = 25
 SPACE = SQUARE_SIZE // 4

 # Colors
 BG_COLOR = (28, 170, 156)
 LINE_COLOR = (23, 145, 135)
 CIRCLE_COLOR = (239, 231, 200)
 CROSS_COLOR = (66, 66, 66)
 TEXT_COLOR = (255, 255, 255)
 BUTTON_COLOR = (66, 66, 66)
 BUTTON_HOVER_COLOR = (100, 100, 100)
 BUTTON_TEXT_COLOR = (255, 255, 255)

 # Screen setup
 screen = pygame.display.set_mode((WIDTH, HEIGHT))
 pygame.display.set_caption('Tic Tac Toe')
 screen.fill(BG_COLOR)

 # Game variables
 board = [[None for _ in range(BOARD_SIZE)] for _ in range(BOARD_SIZE)]
 player = 'X'  # X goes first
 game_over = False
 winner = None


 def draw_lines():
     """Draw the grid lines for the Tic-Tac-Toe board"""
     # Horizontal lines
     for i in range(1, BOARD_SIZE):
         pygame.draw.line(
             screen, LINE_COLOR,
             (0, i * SQUARE_SIZE), (WIDTH, i * SQUARE_SIZE),
             LINE_WIDTH
         )

     # Vertical lines
     for i in range(1, BOARD_SIZE):
         pygame.draw.line(
             screen, LINE_COLOR,
             (i * SQUARE_SIZE, 0), (i * SQUARE_SIZE, WIDTH),
             LINE_WIDTH
         )


 def draw_figures():
     """Draw X's and O's on the board based on the game state"""
     for row in range(BOARD_SIZE):
         for col in range(BOARD_SIZE):
             if board[row][col] == 'O':
                 # Draw O
                 pygame.draw.circle(
                     screen, CIRCLE_COLOR,
                     (col * SQUARE_SIZE + SQUARE_SIZE // 2, row * SQUARE_SIZE + SQUARE_SIZE // 2),
                     CIRCLE_RADIUS, CIRCLE_WIDTH
                 )
             elif board[row][col] == 'X':
                 # Draw X
                 pygame.draw.line(
                     screen, CROSS_COLOR,
                     (col * SQUARE_SIZE + SPACE, row * SQUARE_SIZE + SPACE),
                     (col * SQUARE_SIZE + SQUARE_SIZE - SPACE, row * SQUARE_SIZE + SQUARE_SIZE - SPACE),
                     CROSS_WIDTH
                 )
                 pygame.draw.line(
                     screen, CROSS_COLOR,
                     (col * SQUARE_SIZE + SPACE, row * SQUARE_SIZE + SQUARE_SIZE - SPACE),
                     (col * SQUARE_SIZE + SQUARE_SIZE - SPACE, row * SQUARE_SIZE + SPACE),
                     CROSS_WIDTH
                 )


 def mark_square(row, col, player):
     """Place a player's mark on the board"""
     board[row][col] = player


 def available_square(row, col):
     """Check if a square is available"""
     return board[row][col] is None


 def is_board_full():
     """Check if the board is full"""
     for row in range(BOARD_SIZE):
         for col in range(BOARD_SIZE):
             if board[row][col] is None:
                 return False
     return True


 def check_win():
     """Check if a player has won"""
     # Check rows
     for row in range(BOARD_SIZE):
         if board[row][0] == board[row][1] == board[row][2] and board[row][0] is not None:
             return board[row][0]

     # Check columns
     for col in range(BOARD_SIZE):
         if board[0][col] == board[1][col] == board[2][col] and board[0][col] is not None:
             return board[0][col]

     # Check diagonals
     if board[0][0] == board[1][1] == board[2][2] and board[0][0] is not None:
         return board[0][0]
     if board[0][2] == board[1][1] == board[2][0] and board[0][2] is not None:
         return board[0][2]

     return None


 def draw_status():
     """Draw the status area below the game board"""
     # Clear the status area
     pygame.draw.rect(screen, BG_COLOR, (0, WIDTH, WIDTH, HEIGHT - WIDTH))

     # Create font
     font = pygame.font.SysFont('Arial', 30)

     if game_over:
         if winner:
             text = f"Player {winner} Wins!"
         else:
             text = "It's a Draw!"

         # Draw play again button
         button_rect = pygame.Rect(WIDTH // 2 - 100, WIDTH + 60, 200, 50)
         mouse_pos = pygame.mouse.get_pos()

         if button_rect.collidepoint(mouse_pos):
             pygame.draw.rect(screen, BUTTON_HOVER_COLOR, button_rect)
         else:
             pygame.draw.rect(screen, BUTTON_COLOR, button_rect)

         pygame.draw.rect(screen, (0, 0, 0), button_rect, 2)  # Button border

         button_font = pygame.font.SysFont('Arial', 24)
         button_text = button_font.render("Play Again", True, BUTTON_TEXT_COLOR)
         button_text_rect = button_text.get_rect(center=button_rect.center)
         screen.blit(button_text, button_text_rect)
     else:
         text = f"Player {player}'s Turn"

     # Render and display the status text
     text_surface = font.render(text, True, TEXT_COLOR)
     text_rect = text_surface.get_rect(center=(WIDTH // 2, WIDTH + 30))
     screen.blit(text_surface, text_rect)


 def reset_game():
     """Reset the game to its initial state"""
     global board, player, game_over, winner
     board = [[None for _ in range(BOARD_SIZE)] for _ in range(BOARD_SIZE)]
     player = 'X'
     game_over = False
     winner = None
     screen.fill(BG_COLOR)
     draw_lines()


 # Draw initial board
 draw_lines()

 # Main game loop
 while True:
     for event in pygame.event.get():
         if event.type == pygame.QUIT:
             pygame.quit()
             sys.exit()

         if event.type == pygame.MOUSEBUTTONDOWN and not game_over:
             mouseX, mouseY = event.pos

             # Check if click is within the game board
             if mouseY < WIDTH:
                 clicked_row = mouseY // SQUARE_SIZE
                 clicked_col = mouseX // SQUARE_SIZE

                 if available_square(clicked_row, clicked_col):
                     mark_square(clicked_row, clicked_col, player)

                     # Check for win or draw
                     result = check_win()
                     if result:
                         winner = result
                         game_over = True
                     elif is_board_full():
                         game_over = True

                     # Switch player
                     player = 'O' if player == 'X' else 'X'

         # Check for play again button click
         if event.type == pygame.MOUSEBUTTONDOWN and game_over:
             mouseX, mouseY = event.pos
             button_rect = pygame.Rect(WIDTH // 2 - 100, WIDTH + 60, 200, 50)

             if button_rect.collidepoint((mouseX, mouseY)):
                 reset_game()

         # Reset with 'r' key
         if event.type == pygame.KEYDOWN:
             if event.key == pygame.K_r:
                 reset_game()

     # Update display
     draw_figures()
     draw_status()
     pygame.display.update()
