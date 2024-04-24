import pygame

pygame.init()
 
# Font that is used to render the text
font20 = pygame.font.Font('freesansbold.ttf', 20)
 
# RGB values of standard colors
grey = (1, 30, 47)
METALLICBLUE = (50, 82, 123)
BLACK = (22, 22, 23)

 

# Basic parameters of the screen
WIDTH, HEIGHT = 900, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pong")


# load the image
BG = pygame.image.load("space.jpg")
# reize the image
BG = pygame.transform.scale(BG, (900,600))
BALL = pygame.image.load("starball.png")
BALL = pygame.transform.scale(BALL, (90,60))
clock = pygame.time.Clock()    
FPS = 20



# Striker class
 
 
class Striker:
        # Take the initial position, dimensions, speed and color of the object
    def __init__(self, posx, posy, width, height, speed, color):
        self.posx = posx
        self.posy = posy
        self.width = width
        self.height = height
        self.speed = speed
        self.color = color
        # Rect that is used to control the position and collision of the object
        self.geekRect = pygame.Rect(posx, posy, width, height)
        # Object that is blit on the screen
        self.geek = pygame.draw.rect(screen, self.color, self.geekRect)
 
    # Used to display the object on the screen
    def display(self):
        self.geek = pygame.draw.rect(screen, self.color, self.geekRect)
 
    def update(self, yFac):
        self.posy = self.posy + self.speed*yFac
 
        # Restricting the Striker to be below the top surface of the screen
        if self.posy <= 0:
            self.posy = 0
        # Restricting the Striker to be above the bottom surface of the screen
        elif self.posy + self.height >= HEIGHT:
            self.posy = HEIGHT-self.height
 
        # Updating the rect with the new values
        self.geekRect = (self.posx, self.posy, self.width, self.height)
 
    def displayScore(self, text, score, x, y, color):
        text = font20.render(text+str(score), True, color)
        textRect = text.get_rect()
        textRect.center = (x, y)
 
        screen.blit(text, textRect)
 
    def getRect(self):
        return self.geekRect
 
# Ball class
     
 
class Ball:
    def __init__(self, posx, posy, radius, speed, img):
        self.posx = posx
        self.posy = posy
        self.radius = radius
        self.speed = speed
        self.img = img
        self.xFac = 1
        self.yFac = -1
        self.ball = pygame.Rect((posx,posy,posx+radius, posy+radius))
            # s
            # screen, self.color, (self.posx, self.posy), self.radius)
        self.firstTime = 1 
    def display(self):
        # self.ball = pygame.draw.circle(screen, self.color, (self.posx, self.posy), self.radius)
        screen.blit(self.img, (self.posx, self.posy))

    def update(self):
        self.posx += self.speed*self.xFac
        self.posy += self.speed*self.yFac

        # If the ball hits the top or bottom surfaces, 
        # then the sign of yFac is changed and 
        # it results in a reflection
        if self.posy <= 0 or self.posy >= HEIGHT:
            self.yFac *= -1
 
        if self.posx <= 0:
            self.firstTime = 0
            return 1
        
        elif self.posx >= WIDTH:
            self.firstTime = 0
            return -1
        else:
            return 0
    
    def bounce(self, Striker):
        if(self.posx == Striker.posx):
            if(self.posy == Striker.posy):
                self.xFac = self.xFac * -1    
    def reset(self):
        self.posx = WIDTH//2
        self.posy = HEIGHT//2
        self.xFac *= -1
        self.firstTime = 1
 
    # Used to reflect the ball along the X-axis
    def hit(self):
        self.xFac *= -1
 
    def getRect(self):

        return self.ball
 
# Game Manager
 
 
def main():
    running = True
 
    # Defining the objects
    geek1 = Striker(20, 0, 12, 30, 27,"GREY")
    geek2 = Striker(WIDTH-30, 0, 12, 30, 27, "GREY")
    ball = Ball(WIDTH//2, HEIGHT//9, 8, 9, BALL)
    listOfGeeks = [geek1, geek2]
 
    # Initial parameters of the players
    geek1Score, geek2Score = 0, 0
    geek1YFac, geek2YFac = 0, 0
 
    while running:
        if(ball.posx < 0):
            ball.xFac = ball.xFac * -1
            ball.posx = 1
        if(ball.posx > WIDTH):
            ball.xFac = ball.xFac * -1
            ball.posx = WIDTH-1
        if(ball.posy < 0):
            ball.yFac = ball.yFac * 1
            ball.posy = 1
        if(ball.posy > HEIGHT):
            ball.yFac = ball.yFac * 1
            ball.posy = HEIGHT-1


        screen.fill(METALLICBLUE)

        #blit the image onto the screen at a position
        screen.blit(BG, (0,0))

        # Event handling
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_UP:
                    geek2YFac = -1
                if event.key == pygame.K_DOWN:
                    geek2YFac = 1
                if event.key == pygame.K_w:
                    geek1YFac = -1
                if event.key == pygame.K_s:
                    geek1YFac = 1
            if event.type == pygame.KEYUP:
                if event.key == pygame.K_UP or event.key == pygame.K_DOWN:
                    geek2YFac = 0
                if event.key == pygame.K_w or event.key == pygame.K_s:
                    geek1YFac = 0
 
        # Collision detection
        for geek in listOfGeeks:
            if pygame.Rect.colliderect(ball.getRect(), geek.getRect()):
                ball.hit()
 
        # Updating the objects
        geek1.update(geek1YFac)
        geek2.update(geek2YFac)
        point = ball.update()
        ball.bounce(geek1)
        ball.bounce(geek2)
 
        # -1 -> Geek_1 has scored
        # +1 -> Geek_2 has scored
        #  0 -> None of them scored
        if point == -1:
            geek1Score += 1
        elif point == 1:
            geek2Score += 1
 
        # Someone has scored
        # a point and the ball is out of bounds.
        # So, we reset it's position
        # if point:   
            # ball.reset()
 
        # Displaying the objects on the screen
        geek1.display()
        geek2.display()
        ball.display()
 
        # Displaying the scores of the players
        geek1.displayScore("Geek_1 : ", 
                           geek1Score, 100, 20, "grey")
        geek2.displayScore("Geek_2 : ", 
                           geek2Score, WIDTH-100, 20, "grey")
 
        pygame.display.update()
        clock.tick(FPS)     
 
 
if __name__ == "__main__":
    main()
    pygame.quit()