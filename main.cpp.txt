#include <graphics.h>
#include <conio.h>
#include <cmath>
#include <cstdlib>
#include <ctime>

#define WIDTH 800
#define HEIGHT 600
#define SHIP_SIZE 25
#define BULLET_SPEED 10
#define ENEMY_SIZE 30
#define PI 3.14159265

// Define the initial enemy speed and speed increase intervals
#define INITIAL_ENEMY_SPEED 5
#define SPEED_INCREASE_INTERVAL_1 10
#define SPEED_INCREASE_INTERVAL_2 20
#define SPEED_INCREASE_INTERVAL_3 50
#define SPEED_INCREASE_INTERVAL_4 70

struct Ship {
    int x, y;
};

struct Bullet {
    int x, y;
    bool active;
};

struct Enemy {
    int x, y;
    bool active;
};

Ship player;
Bullet bullets[1000];
Enemy enemy;
int score = 0;
int enemySpeed = INITIAL_ENEMY_SPEED; // Initialize enemy speed
int level = 1; // Initialize level

void initialize() {
    initwindow(WIDTH, HEIGHT, "Space Shooter");
    player.x = WIDTH / 2;
    player.y = HEIGHT - 50;
    enemy.active = false;

    srand(time(NULL)); // Seed the random number generator
    for (int i = 0; i < 10; i++) {
        bullets[i].active = false;
    }

    // Display instructions
    setcolor(WHITE);
    settextstyle(DEFAULT_FONT, HORIZ_DIR, 2); // Font size for instructions
    outtextxy(50, HEIGHT / 2 - 50, "Press SPACE to shoot.");
    outtextxy(50, HEIGHT / 2, "Press A to move left, D to move right.");
    outtextxy(50, HEIGHT / 2 + 50, "Press any key to start the game.");
    getch(); // Wait for any key press to start the game
    cleardevice(); // Clear the screen before starting the game
}

void drawShip(int x, int y) {
    int points[] = { x, y - SHIP_SIZE, x + SHIP_SIZE, y + SHIP_SIZE, x - SHIP_SIZE, y + SHIP_SIZE };
    setfillstyle(SOLID_FILL, WHITE); // Fill ship with white color
    fillpoly(3, points);
}

void drawEnemy(int x, int y) {
    int points[] = { x + ENEMY_SIZE, y,
                    x + ENEMY_SIZE * 0.5, y + ENEMY_SIZE * sqrt(3) / 2,
                    x - ENEMY_SIZE * 0.5, y + ENEMY_SIZE * sqrt(3) / 2,
                    x - ENEMY_SIZE, y,
                    x - ENEMY_SIZE * 0.5, y - ENEMY_SIZE * sqrt(3) / 2,
                    x + ENEMY_SIZE * 0.5, y - ENEMY_SIZE * sqrt(3) / 2 };
    setcolor(RED);
    setfillstyle(SOLID_FILL, RED); // Fill enemy with red color
    fillpoly(6, points);
}

void drawBullet(int x, int y) {
    setcolor(GREEN);
    setfillstyle(SOLID_FILL, GREEN); // Fill bullet with green color
    rectangle(x, y, x + 5, y + 15);
    floodfill(x + 2, y + 7, GREEN); // Fill the interior of the rectangle with green
}

void drawScore() {
    setcolor(WHITE);
    settextstyle(DEFAULT_FONT, HORIZ_DIR, 3); // Default font size
    char scoreStr[20];
    sprintf(scoreStr, "Score: %d", score);
    outtextxy(10, 10, scoreStr);

    char levelStr[20];
    sprintf(levelStr, "Level: %d", enemySpeed - INITIAL_ENEMY_SPEED + 1); // Calculate level based on enemy speed
    outtextxy(10, 40, levelStr); // Draw level below the score
}



void drawGameOver() {
    setcolor(RED);
    settextstyle(DEFAULT_FONT, HORIZ_DIR, 4); // Reduced font size for game over message
    int textWidth = textwidth("GAME OVER");
    int textHeight = textheight("GAME OVER");
    outtextxy((WIDTH - textWidth) / 2, (HEIGHT - textHeight) / 2, "GAME OVER");

    setcolor(WHITE);
    settextstyle(DEFAULT_FONT, HORIZ_DIR, 3); // Default font size for score
    char scoreStr[20];
    sprintf(scoreStr, "Score: %d", score);
    outtextxy((WIDTH - textwidth(scoreStr)) / 2, (HEIGHT + textHeight) / 2, scoreStr);
}

void shootBullet() {
    static int lastShootTime = 0; // Keep track of the last shoot time
    int currentTime = clock(); // Get current time
    if (currentTime - lastShootTime < 200) // Limit shooting rate (200 milliseconds per bullet)
        return;

    for (int i = 0; i < 10; i++) {
        if (!bullets[i].active) {
            // Calculate the position of the tip of the ship
            int tipX = player.x - 2;
            int tipY = player.y - SHIP_SIZE;

            bullets[i].x = tipX; // Set bullet x-coordinate to the tip's x-coordinate
            bullets[i].y = tipY; // Set bullet y-coordinate to the tip's y-coordinate
            bullets[i].active = true;
            lastShootTime = currentTime; // Update last shoot time
            break;
        }
    }
}

void spawnEnemy() {
    if (!enemy.active) {
        enemy.x = rand() % (WIDTH - 2 * ENEMY_SIZE) + ENEMY_SIZE;
        enemy.y = 0;
        enemy.active = true;
    }
}

void moveBullets() {
    for (int i = 0; i < 10; i++) {
        if (bullets[i].active) {
            bullets[i].y -= BULLET_SPEED;
            if (bullets[i].y < 0) {
                bullets[i].active = false;
            }
        }
    }
}

void moveEnemy() {
    if (enemy.active) {
        enemy.y += enemySpeed;
        if (enemy.y > HEIGHT) {
            enemy.active = false;
        }
    }
}

bool isCollision(int x1, int y1, int w1, int h1, int x2, int y2, int w2, int h2) {
    return (x1 < x2 + w2 && x1 + w1 > x2 && y1 < y2 + h2 && y1 + h1 > y2);
}

void checkCollisions() {
    if (enemy.active) {
        for (int j = 0; j < 10; j++) {
            if (bullets[j].active) {
                if (isCollision(bullets[j].x, bullets[j].y, 5, 15, enemy.x - ENEMY_SIZE, enemy.y - ENEMY_SIZE, 2 * ENEMY_SIZE, 2 * ENEMY_SIZE)) {
                    bullets[j].active = false;
                    enemy.active = false;
                    score++; // Increase score when hitting enemy
                }
            }
        }
        if (isCollision(player.x - SHIP_SIZE/2, player.y - SHIP_SIZE/2, SHIP_SIZE, SHIP_SIZE, enemy.x - ENEMY_SIZE, enemy.y - ENEMY_SIZE, 2 * ENEMY_SIZE, 2 * ENEMY_SIZE)) {
            closegraph();
            initwindow(WIDTH, HEIGHT);
            drawGameOver();
            getch();
            closegraph();
            exit(0);
        }
    }
}

void checkEnemyPassed() {
    if (enemy.y > HEIGHT) {
        closegraph();
        initwindow(WIDTH, HEIGHT);
        drawGameOver();
        getch();
        closegraph();
        exit(0);
    }
}

void checkScore() {
    if (score >= SPEED_INCREASE_INTERVAL_1 && score < SPEED_INCREASE_INTERVAL_2) {
        enemySpeed = INITIAL_ENEMY_SPEED + 1;
        level = 2; // Update level
    } else if (score >= SPEED_INCREASE_INTERVAL_2 && score < SPEED_INCREASE_INTERVAL_3) {
        enemySpeed = INITIAL_ENEMY_SPEED + 2;
        level = 3; // Update level
    } else if (score >= SPEED_INCREASE_INTERVAL_3 && score < SPEED_INCREASE_INTERVAL_4) {
        enemySpeed = INITIAL_ENEMY_SPEED + 3;
        level = 4; // Update level
    } else if (score >= SPEED_INCREASE_INTERVAL_4) {
        enemySpeed = INITIAL_ENEMY_SPEED + 4;
        level = 5; // Update level
    }
}

int main() {
    initialize();

    while (true) {
        cleardevice();

        if (kbhit()) {
            char key = getch();
            if (key == 'a' && player.x - ((SHIP_SIZE/2) + 20) > 0) { // Check if moving left won't go out of the screen
                player.x -= 10;
            }
            if (key == 'd' && player.x + ((SHIP_SIZE/2) + 20) < WIDTH) { // Check if moving right won't go out of the screen
                player.x += 10;
            }
            if (key == ' ') {
                shootBullet();
            }
        }

        if (!enemy.active) {
            spawnEnemy();
        }

        moveBullets();
        moveEnemy();
        checkCollisions();
        checkEnemyPassed(); // Check if enemy has passed the screen
        checkScore(); // Check the score for speed increase

        drawShip(player.x, player.y);
        for (int i = 0; i < 10; i++) {
            if (bullets[i].active) {
                drawBullet(bullets[i].x, bullets[i].y);
            }
        }
        if (enemy.active) {
            drawEnemy(enemy.x, enemy.y);
        }

        drawScore(); // Draw score
        delay(30);
    }

    closegraph();
    return 0;
}
