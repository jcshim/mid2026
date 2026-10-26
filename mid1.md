# 1번 답안
## 무지개 블록깨기 게임을 C언어로 작성하시오. (그래픽 라이브러리 활용)
## 나의 프람프트: 
무지개 블록깨기 게임을 C언어로 작성하시오. 
(그래픽 raylib 최신 버전 라이브러리 활용), 
visual studio community 최근 버전에서 nuget에서 설치해서 사용하고 싶어, 한글은 절대 사용하지 말 것) 
반드시 작동이 되어야 하고, 5회 실패시 다시 시작하도록 해줘, 코드는 간결하게.
---
```
#include "raylib.h"

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600
#define COLS 10
#define ROWS 6
#define MAX_LIVES 5

typedef struct {
    Rectangle rect;
    Color color;
    bool active;
} Block;

int main(void) {
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Brick Breaker");
    SetTargetFPS(60);

    Color rainbowColors[ROWS] = { RED, ORANGE, YELLOW, GREEN, BLUE, PURPLE };

    // Game state variables
    int lives = MAX_LIVES;
    int score = 0;
    bool gameOver = false;

    // Paddle and Ball
    Rectangle paddle = { SCREEN_WIDTH / 2.0f - 60, SCREEN_HEIGHT - 40, 120, 15 };
    Vector2 ball = { SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
    Vector2 ballSpeed = { 5.0f, -5.0f };
    float ballRadius = 8.0f;

    // Blocks setup
    Block blocks[ROWS][COLS];
    float blockWidth = (float)SCREEN_WIDTH / COLS;
    float blockHeight = 20.0f;

    for (int r = 0; r < ROWS; r++) {
        for (int c = 0; c < COLS; c++) {
            blocks[r][c].rect = (Rectangle){ c * blockWidth + 2, r * blockHeight + 40, blockWidth - 4, blockHeight - 4 };
            blocks[r][c].color = rainbowColors[r];
            blocks[r][c].active = true;
        }
    }

    while (!WindowShouldClose()) {
        // --- Update ---
        if (!gameOver) {
            // Paddle movement
            if (IsKeyDown(KEY_LEFT) && paddle.x > 0) paddle.x -= 8.0f;
            if (IsKeyDown(KEY_RIGHT) && paddle.x < SCREEN_WIDTH - paddle.width) paddle.x += 8.0f;

            // Ball movement
            ball.x += ballSpeed.x;
            ball.y += ballSpeed.y;

            // Wall collisions
            if (ball.x - ballRadius <= 0 || ball.x + ballRadius >= SCREEN_WIDTH) ballSpeed.x *= -1;
            if (ball.y - ballRadius <= 0) ballSpeed.y *= -1;

            // Paddle collision
            if (CheckCollisionCircleRec(ball, ballRadius, paddle)) {
                ballSpeed.y = -4fabs(ballSpeed.y);
                ballSpeed.x = (ball.x - (paddle.x + paddle.width / 2.0f)) * 0.15f;
            }

            // Block collisions
            for (int r = 0; r < ROWS; r++) {
                for (int c = 0; c < COLS; c++) {
                    if (blocks[r][c].active && CheckCollisionCircleRec(ball, ballRadius, blocks[r][c].rect)) {
                        blocks[r][c].active = false;
                        ballSpeed.y *= -1;
                        score += 10;
                        break;
                    }
                }
            }

            // Bottom boundary (Life lost)
            if (ball.y + ballRadius >= SCREEN_HEIGHT) {
                lives--;
                if (lives <= 0) {
                    gameOver = true;
                } else {
                    // Reset positions
                    ball = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
                    ballSpeed = (Vector2){ 5.0f, -5.0f };
                    paddle.x = SCREEN_WIDTH / 2.0f - 60;
                }
            }
        } else {
            // Restart game on SPACE
            if (IsKeyPressed(KEY_SPACE)) {
                lives = MAX_LIVES;
                score = 0;
                gameOver = false;
                ball = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
                ballSpeed = (Vector2){ 5.0f, -5.0f };
                paddle.x = SCREEN_WIDTH / 2.0f - 60;
                for (int r = 0; r < ROWS; r++) {
                    for (int c = 0; c < COLS; c++) {
                        blocks[r][c].active = true;
                    }
                }
            }
        }

        // --- Draw ---
        BeginDrawing();
        ClearBackground(BLACK);

        if (!gameOver) {
            // Draw blocks
            for (int r = 0; r < ROWS; r++) {
                for (int c = 0; c < COLS; c++) {
                    if (blocks[r][c].active) {
                        DrawRectangleRec(blocks[r][c].rect, blocks[r][c].color);
                    }
                }
            }

            // Draw paddle and ball
            DrawRectangleRec(paddle, WHITE);
            DrawCircleV(ball, ballRadius, WHITE);

            // Draw HUD
            DrawText(TextFormat("SCORE: %d", score), 20, 10, 20, RAYWHITE);
            DrawText(TextFormat("LIVES: %d", lives), SCREEN_WIDTH - 120, 10, 20, RED);
        } else {
            // Game Over Screen
            DrawText("GAME OVER", SCREEN_WIDTH / 2 - 140, SCREEN_HEIGHT / 2 - 40, 50, RED);
            DrawText("Press SPACE to Restart", SCREEN_WIDTH / 2 - 150, SCREEN_HEIGHT / 2 + 20, 20, RAYWHITE);
        }

        EndDrawing();
    }
```
<img width="799" height="621" alt="image" src="https://github.com/user-attachments/assets/88b07293-5a4d-428f-b99f-1a6d536cfde1" />

    CloseWindow();
    return 0;
}
