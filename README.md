#include <SFML/Graphics.hpp>  // Voor alles wat met grafische objecten te maken heeft, zoals Color, Text, enz.
#include <SFML/Window.hpp>     // Voor vensterfunctionaliteit
#include <SFML/Audio.hpp>      // Als je audio nodig hebt
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include "Game.h"

#include "Enemy.h"
#include "Player.h"




// Game class

int main() {
    Game game;
    game.run();

    return 0;
}

#pragma once
#include <SFML/Graphics/RectangleShape.hpp>
#include <SFML/System/Vector2.hpp>
#include <SFML/System/Time.hpp>
#include <SFML/Window/Keyboard.hpp>

class Player {
public:
    sf::RectangleShape shape;
    sf::Vector2f velocity;
    float speed = 100.0f;
    float runSpeed = 300.0f;
    bool isJumping = false;
    bool isRunning = false;
    float gravity = 0.5f;
    float jumpStrength = -500.0f;

    Player() {
        shape.setSize(sf::Vector2f(50.0f, 100.0f));
        shape.setFillColor(sf::Color::Green);
        shape.setPosition(100.0f, 400.0f);
    }

    void move(sf::Time deltaTime, bool isPaused) {
        if (isPaused) {
            velocity = sf::Vector2f(0.0f, 0.0f);
            return;
        }

        velocity.x = 0.0f;

        isRunning = sf::Keyboard::isKeyPressed(sf::Keyboard::LShift);

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left)) {
            velocity.x = isRunning ? -runSpeed : -speed;
        }
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right)) {
            velocity.x = isRunning ? runSpeed : speed;
        }

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) && !isJumping) {
            velocity.y = jumpStrength;
            isJumping = true;
        }

        if (isJumping) {
            velocity.y += gravity;
        }

        shape.move(velocity * deltaTime.asSeconds());
    }

    void land() {
        isJumping = false;
        velocity.y = 0;
    }
};
#pragma once
#include <SFML/Graphics/RectangleShape.hpp>
#include <SFML/System/Vector2.hpp>
#include <SFML/System/Time.hpp>
#include <SFML/Window/Keyboard.hpp>

class Player {
public:
    sf::RectangleShape shape;
    sf::Vector2f velocity;
    float speed = 100.0f;
    float runSpeed = 300.0f;
    bool isJumping = false;
    bool isRunning = false;
    float gravity = 0.5f;
    float jumpStrength = -500.0f;

    Player() {
        shape.setSize(sf::Vector2f(50.0f, 100.0f));
        shape.setFillColor(sf::Color::Green);
        shape.setPosition(100.0f, 400.0f);
    }

    void move(sf::Time deltaTime, bool isPaused) {
        if (isPaused) {
            velocity = sf::Vector2f(0.0f, 0.0f);
            return;
        }

        velocity.x = 0.0f;

        isRunning = sf::Keyboard::isKeyPressed(sf::Keyboard::LShift);

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Left)) {
            velocity.x = isRunning ? -runSpeed : -speed;
        }
        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Right)) {
            velocity.x = isRunning ? runSpeed : speed;
        }

        if (sf::Keyboard::isKeyPressed(sf::Keyboard::Up) && !isJumping) {
            velocity.y = jumpStrength;
            isJumping = true;
        }

        if (isJumping) {
            velocity.y += gravity;
        }

        shape.move(velocity * deltaTime.asSeconds());
    }

    void land() {
        isJumping = false;
        velocity.y = 0;
    }
};
#pragma once
#include <SFML/Graphics.hpp>
#include "Player.h"
#include "Enemy.h"
#include <vector>
#include <iostream>

class Game {
private:
    sf::RenderWindow window;
    sf::View view;
    Player player;
    std::vector<sf::RectangleShape> platforms;
    std::vector<Enemy> enemies;
    bool isPaused = false;
    bool hasWon = false;
    int score = 0;
    int enemiesKilled = 0;
    sf::RectangleShape pauseOverlay;
    sf::Clock clock;
    sf::Font font;

    const int winningKillCount = 5;
    const sf::Vector2f cameraOffset = { 200.0f, 300.0f };

public:
    Game();
    void run();

private:
    void setupPlatforms();
    void setupEnemies();
    void setupOverlay();
    void addPlatform(const sf::Vector2f& size, const sf::Vector2f& position);
    void handleEvents();
    void update();
    void checkCollisions();
    void updateCamera();
    void draw();
    void drawWinMessage();
};
#include "Game.h"

Game::Game() : window(sf::VideoMode(800, 600), "Parkour Game"), view(sf::FloatRect(0, 0, 800, 600)) {
    setupPlatforms();
    setupEnemies();
    setupOverlay();

    if (!font.loadFromFile("arial.ttf")) {
        std::cerr << "Error: Could not load font\n";
    }
}

void Game::run() {
    while (window.isOpen()) {
        handleEvents();
        update();
        draw();
    }
}

void Game::setupPlatforms() {
    addPlatform({ 1600.0f, 50.0f }, { 0.0f, 550.0f });
    addPlatform({ 200.0f, 50.0f }, { 400.0f, 450.0f });
    addPlatform({ 300.0f, 50.0f }, { 800.0f, 400.0f });
    addPlatform({ 400.0f, 50.0f }, { 1300.0f, 350.0f });
}

void Game::setupEnemies() {
    enemies.emplace_back(200.0f, 450.0f);
    enemies.emplace_back(700.0f, 400.0f);
    enemies.emplace_back(1100.0f, 300.0f);
    enemies.emplace_back(1500.0f, 200.0f);
    enemies.emplace_back(1800.0f, 450.0f);
}

void Game::setupOverlay() {
    pauseOverlay.setSize(sf::Vector2f(800.0f, 600.0f));
    pauseOverlay.setFillColor(sf::Color(0, 0, 0, 150));
}

void Game::addPlatform(const sf::Vector2f& size, const sf::Vector2f& position) {
    sf::RectangleShape platform(size);
    platform.setFillColor(sf::Color::Blue);
    platform.setPosition(position);
    platforms.push_back(platform);
}

void Game::handleEvents() {
    sf::Event event;
    while (window.pollEvent(event)) {
        if (event.type == sf::Event::Closed)
            window.close();
        if (event.type == sf::Event::KeyPressed && event.key.code == sf::Keyboard::Escape) {
            isPaused = !isPaused;
        }
    }
}

void Game::update() {
    if (isPaused || hasWon) return;

    sf::Time deltaTime = clock.restart();
    player.move(deltaTime, isPaused);

    checkCollisions();
    updateCamera();
}

void Game::checkCollisions() {
    bool onGround = false;
    for (auto& platform : platforms) {
        if (player.shape.getGlobalBounds().intersects(platform.getGlobalBounds())) {
            if (player.shape.getPosition().y + player.shape.getSize().y <= platform.getPosition().y + 1) {
                player.shape.setPosition(player.shape.getPosition().x, platform.getPosition().y - player.shape.getSize().y);
                player.land();
                onGround = true;
                break;
            }
        }
    }

    if (!onGround) {
        player.velocity.y += player.gravity;
    }

    for (auto& enemy : enemies) {
        enemy.moveTowardsPlayer(player);

        if (enemy.isAlive && player.shape.getGlobalBounds().intersects(enemy.shape.getGlobalBounds())) {
            enemy.isAlive = false;
            enemiesKilled++;
            score += 10;

            if (enemiesKilled >= winningKillCount) {
                hasWon = true;
            }
        }
    }
}

void Game::updateCamera() {
    view.setCenter(player.shape.getPosition().x + cameraOffset.x, cameraOffset.y);
    window.setView(view);
}

void Game::draw() {
    window.clear();

    for (auto& platform : platforms) {
        window.draw(platform);
    }

    for (auto& enemy : enemies) {
        enemy.draw(window);
    }

    window.draw(player.shape);

    if (isPaused) {
        window.draw(pauseOverlay);
    }

    if (hasWon) {
        drawWinMessage();
    }

    window.display();
}

void Game::drawWinMessage() {
    sf::Text winText("You Win!", font, 50);
    winText.setFillColor(sf::Color::Yellow);
    winText.setPosition(view.getCenter().x - 100, view.getCenter().y - 50);
    window.draw(winText);
}
#pragma once
#include <SFML/System/Vector2.hpp>
#include <SFML/Graphics/RenderWindow.hpp>
#include <SFML/Graphics/RectangleShape.hpp>
#include <cmath> // Voor berekeningen zoals std::sqrt()

// Forward declaration van Player
class Player;

class Enemy {
public:
    sf::RectangleShape shape;
    bool isAlive = true;
    float speed = 2.0f;

    Enemy(float x, float y);

    void moveTowardsPlayer(const Player& player);
    void draw(sf::RenderWindow& window);
};
#include "Enemy.h"
#include "Player.h" // Hier wordt de volledige definitie van Player ingeladen

Enemy::Enemy(float x, float y) {
    shape.setSize(sf::Vector2f(50.0f, 100.0f));
    shape.setFillColor(sf::Color::Red);
    shape.setPosition(x, y);
}

void Enemy::moveTowardsPlayer(const Player& player) {
    if (!isAlive) return;

    sf::Vector2f direction = player.shape.getPosition() - shape.getPosition();
    float distance = std::sqrt(direction.x * direction.x + direction.y * direction.y);

    if (distance < 300.0f) {
        direction /= distance; // Normaliseer richting
        shape.move(direction * speed);
    }
}

void Enemy::draw(sf::RenderWindow& window) {
    if (isAlive) {
        window.draw(shape);
    }
}
