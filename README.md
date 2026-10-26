# -12
중간고사 제출을 위한 저장소

무지개 블록깨기 게임을 c언어로 작성하시오.
#include "raylib.h"

#define SCREEN_WIDTH 800
#define SCREEN_HEIGHT 600
#define ROWS 5
#define COLS 10

void InitBlocks(int blocks[ROWS][COLS]) {
    for (int i = 0; i < ROWS; i++) {
        for (int j = 0; j < COLS; j++) {
            blocks[i][j] = 1;
        }
    }
}

int main(void) {
    InitWindow(SCREEN_WIDTH, SCREEN_HEIGHT, "Rainbow Breakout");
    SetTargetFPS(60);

    Rectangle paddle = { SCREEN_WIDTH / 2.0f - 50, SCREEN_HEIGHT - 40, 100, 15 };
    Vector2 ballPos = { SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
    Vector2 ballVel = { 5.0f, -5.0f };
    float ballRadius = 8.0f;

    int blocks[ROWS][COLS];
    InitBlocks(blocks);

    int lives = 5;
    int score = 0;

    Color rainbow[ROWS] = { RED, ORANGE, YELLOW, GREEN, BLUE };

    while (!WindowShouldClose()) {
        // Paddle movement
        if (IsKeyDown(KEY_LEFT) || IsKeyDown(KEY_A)) paddle.x -= 7.0f;
        if (IsKeyDown(KEY_RIGHT) || IsKeyDown(KEY_D)) paddle.x += 7.0f;
        if (paddle.x < 0) paddle.x = 0;
        if (paddle.x + paddle.width > SCREEN_WIDTH) paddle.x = SCREEN_WIDTH - paddle.width;

        // Ball movement
        ballPos.x += ballVel.x;
        ballPos.y += ballVel.y;

        // Wall collisions
        if (ballPos.x - ballRadius <= 0 || ballPos.x + ballRadius >= SCREEN_WIDTH) ballVel.x *= -1;
        if (ballPos.y - ballRadius <= 0) ballVel.y *= -1;

        // Bottom collision (Lose life / 5 failures reset)
        if (ballPos.y >= SCREEN_HEIGHT) {
            lives--;
            if (lives <= 0) {
                lives = 5;
                score = 0;
                InitBlocks(blocks);
            }
            ballPos = (Vector2){ SCREEN_WIDTH / 2.0f, SCREEN_HEIGHT / 2.0f };
            ballVel = (Vector2){ 5.0f, -5.0f };
        }

        // Paddle collision
        Rectangle ballRect = { ballPos.x - ballRadius, ballPos.y - ballRadius, ballRadius * 2, ballRadius * 2 };
        if (CheckCollisionRecs(ballRect, paddle)) {
            ballVel.y *= -1;
            float hitFactor = (ballPos.x - (paddle.x + paddle.width / 2.0f)) / (paddle.width / 2.0f);
            ballVel.x = hitFactor * 6.0f;
        }

        // Block collisions
        float blockWidth = (float)SCREEN_WIDTH / COLS;
        float blockHeight = 25.0f;
        for (int i = 0; i < ROWS; i++) {
            for (int j = 0; j < COLS; j++) {
                if (blocks[i][j] == 1) {
                    Rectangle blockRect = { j * blockWidth, i * blockHeight + 50, blockWidth - 2, blockHeight - 2 };
                    if (CheckCollisionRecs(ballRect, blockRect)) {
                        blocks[i][j] = 0;
                        ballVel.y *= -1;
                        score += 10;
                    }
                }
            }
        }

        // Render
        BeginDrawing();
        ClearBackground(RAYWHITE);

        DrawRectangleRec(paddle, DARKGRAY);
        DrawCircleV(ballPos, ballRadius, BLACK);

        for (int i = 0; i < ROWS; i++) {
            for (int j = 0; j < COLS; j++) {
                if (blocks[i][j] == 1) {
                    Rectangle blockRect = { j * blockWidth, i * blockHeight + 50, blockWidth - 2, blockHeight - 2 };
                    DrawRectangleRec(blockRect, rainbow[i]);
                }
            }
        }

        DrawText(TextFormat("Score: %d", score), 10, 10, 20, DARKGRAY);
        DrawText(TextFormat("Lives: %d", lives), SCREEN_WIDTH - 110, 10, 20, DARKGRAY);

        EndDrawing();
    }

    CloseWindow();
    return 0;
}

