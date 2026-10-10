# Space-Shooter-Game
The game has a player and non killable objects with an enemy kill 10 enemies to win kill and avoid all enemies to win 
# Create your own shooter
from pygame import *
from random import randint

win_width = 700
win_height = 500
window = display.set_mode((win_width, win_height))
display.set_caption("Catch")
background = transform.scale(image.load("galaxy.jpg"), (win_width, win_height))
timer = 0

# number of enemies that have escaped
lost = 0

# number of enemies killed
score = 0 

game = True
finish = False
clock = time.Clock()
FPS = 60

# Music upload
init()
mixer.init()
mixer.music.load("space.ogg")
mixer.music.play()
fire_sound = mixer.Sound('fire.ogg')
font.init()
bigstyle = font.Font(None, 36)
win = bigstyle.render("YOU WIN", True, (255, 215, 0))
text_lose = bigstyle.render("YOU LOSE", True, (255, 0, 0))

smallstyle = font.Font(None, 20)

class GameSprite(sprite.Sprite):
    #class constructor
    def __init__(self, player_image, player_x, player_y, player_speed):
        super().__init__()
        #every sprite must store the image property
        self.image = transform.scale(image.load(player_image), (55,55))
        self.speed = player_speed

        #every sprite must have the rect property  - the rectangle it is fitted in
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
    
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))

class Player(GameSprite):
    def update(self):
        key_pressed = key.get_pressed()
        if key_pressed[K_a] and self.rect.x > 5:
            self.rect.x -= self.speed
        if key_pressed[K_d] and self.rect.x < win_width - 80:
            self.rect.x += self.speed
        if key_pressed[K_SPACE]:
            self.fire()

    def fire(self):
        player_projectile = PlayerProjectile("asteroid.png", self.rect.centerx - 27.5, self.rect.top - 30, 15 )
        groupAsteroids.add(player_projectile)
        fire_sound.play()

player = Player("bullet.png", 5, win_height - 80, 12)

class PlayerProjectile(GameSprite):
    def update(self):
        if self.rect.y >= 0:
            self.rect.y -= self.speed
        else:
            self.kill()

class Monster(GameSprite):
    def update(self):
        global lost
        if self.rect.y <= 470:
            self.side = "down"
            self.rect.y += self.speed
        else:
            self.rect.y = 0
            self.rect.x = randint(80, win_width - 40)
            lost += 1

class Asteroid(GameSprite):
    def update(self):
        global lost
        if self.rect.y <= 470:
            self.side = "down"
            self.rect.y += self.speed
        else:
            self.rect.y = 0
            self.rect.x = randint(80, win_width - 40)
            lost += 1

class Bullet(GameSprite):
    def update(self):
        if self.rect.y <= 470:
            self.side = "down"
            self.rect.y += self.speed
        else:
            self.rect.y = 0
            self.rect.x = randint(60, win_width - 30)

groupship = sprite.Group()
groupMonsters = sprite.Group()
groupAsteroids = sprite.Group()

for i in range(0, 6):
    ship = Monster("rocket.png", randint(60, win_width - 30), 60, 3)
    groupship.add(ship)

for i in range(0, 3):
    monster1 = Monster("ufo.png", randint(80, win_width - 40), 60, 3)
    groupMonsters.add(monster1)

while game:
    for e in event.get():
        if e.type == QUIT:
            game = False

    if finish != True:
        window.blit(background, (0, 0))
        player.update()
        player.reset()
        groupship.draw(window)
        groupship.update()
        groupMonsters.draw(window)
        groupMonsters.update()
        groupAsteroids.draw(window)
        groupAsteroids.update()
        
        scoreText = smallstyle.render(("Enemies Killed: " + str(score)), True, (255, 50, 50))
        window.blit(scoreText, (50, 20))
        
        bullet_lists = sprite.groupcollide(
            groupMonsters, groupAsteroids, True, True
        )
        for thing in bullet_lists:
            score += 1 
            monster1 = Monster("ufo.png", randint(80, win_width - 40), 60, 3)
            groupMonsters.add(monster1)
            
        player_lists = sprite.spritecollide(
            player, groupMonsters, False
        )
    
        # Win Score reaches 10
        if score >= 10:
            finish = True
        # Lose display text
        elif len(player_lists) > 0:
            finish = True
            
    else:        
        #fixed score timer/ win shall now display.
        if score >= 10:
            window.blit(win, (280, 230))
        else:
            window.blit(text_lose, (280, 230))

    display.update()
    clock.tick(FPS)
