# Bass Runner — Flipper Zero Mini Game

A simple endless runner game for the Flipper Zero where you control a bass fish dodging hooks and boats.

## Features

* Endless side-scrolling gameplay
* Increasing speed over time
* Score counter
* Game over + restart
* Made for the Flipper Zero using the Furi API

---

# File Structure

Create a folder:

```text
applications_user/bass_runner/
```

Inside it, create these files:

```text
bass_runner.c
application.fam
```

---

# application.fam

```c
App(
    appid="bass_runner",
    name="Bass Runner",
    apptype=FlipperAppType.EXTERNAL,
    entry_point="bass_runner_app",
    cdefines=["APP_BASS_RUNNER"],
    requires=["gui"],
    stack_size=2 * 1024,
    order=30,
    fap_category="Games",
)
```

---

# bass_runner.c

```c
#include <furi.h>
#include <gui/gui.h>
#include <input/input.h>
#include <stdlib.h>

#define SCREEN_W 128
#define SCREEN_H 64

typedef struct {
    int x;
    int y;
    int active;
} Obstacle;

typedef struct {
    int fish_y;
    int velocity;
    int score;
    int speed;
    bool running;
    Obstacle obstacle;
} GameState;

static void spawn_obstacle(GameState* game) {
    game->obstacle.x = SCREEN_W;
    game->obstacle.y = 44;
    game->obstacle.active = 1;
}

static void game_draw(Canvas* canvas, GameState* game) {
    canvas_clear(canvas);

    canvas_set_font(canvas, FontPrimary);

    char score_text[32];
    snprintf(score_text, sizeof(score_text), "Score: %d", game->score);
    canvas_draw_str(canvas, 2, 10, score_text);

    // Ground
    canvas_draw_line(canvas, 0, 54, 128, 54);

    // Fish
    canvas_draw_disc(canvas, 20, game->fish_y, 4);
    canvas_draw_line(canvas, 16, game->fish_y, 10, game->fish_y - 3);
    canvas_draw_line(canvas, 16, game->fish_y, 10, game->fish_y + 3);

    // Obstacle (hook)
    if(game->obstacle.active) {
        canvas_draw_box(canvas, game->obstacle.x, game->obstacle.y, 5, 10);
        canvas_draw_line(canvas, game->obstacle.x + 2, game->obstacle.y, game->obstacle.x + 2, game->obstacle.y - 6);
    }

    if(!game->running) {
        canvas_set_font(canvas, FontPrimary);
        canvas_draw_str(canvas, 30, 28, "GAME OVER");
        canvas_set_font(canvas, FontSecondary);
        canvas_draw_str(canvas, 12, 42, "Press OK to restart");
    }
}

static void game_update(GameState* game) {
    if(!game->running) return;

    game->velocity += 1;
    game->fish_y += game->velocity;

    if(game->fish_y > 50) {
        game->fish_y = 50;
        game->velocity = 0;
    }

    if(game->fish_y < 8) {
        game->fish_y = 8;
    }

    if(game->obstacle.active) {
        game->obstacle.x -= game->speed;

        if(game->obstacle.x < -10) {
            game->score++;

            if(game->score % 5 == 0) {
                game->speed++;
            }

            spawn_obstacle(game);
        }
    }

    // Collision
    if(game->obstacle.x < 25 && game->obstacle.x > 12) {
        if(game->fish_y > 38) {
            game->running = false;
        }
    }
}

static void input_callback(InputEvent* input_event, void* ctx) {
    FuriMessageQueue* event_queue = ctx;
    furi_message_queue_put(event_queue, input_event, FuriWaitForever);
}

static void render_callback(Canvas* canvas, void* ctx) {
    GameState* game = ctx;
    game_draw(canvas, game);
}

int32_t bass_runner_app(void* p) {
    UNUSED(p);

    GameState game = {
        .fish_y = 30,
        .velocity = 0,
        .score = 0,
        .speed = 2,
        .running = true,
    };

    spawn_obstacle(&game);

    FuriMessageQueue* event_queue = furi_message_queue_alloc(8, sizeof(InputEvent));

    ViewPort* view_port = view_port_alloc();
    view_port_draw_callback_set(view_port, render_callback, &game);
    view_port_input_callback_set(view_port, input_callback, event_queue);

    Gui* gui = furi_record_open(RECORD_GUI);
    gui_add_view_port(gui, view_port, GuiLayerFullscreen);

    while(1) {
        InputEvent event;

        if(furi_message_queue_get(event_queue, &event, 50) == FuriStatusOk) {
            if(event.type == InputTypePress) {
                if(event.key == InputKeyBack) {
                    break;
                }

                if(event.key == InputKeyOk) {
                    if(game.running) {
                        game.velocity = -5;
                    } else {
                        game.fish_y = 30;
                        game.velocity = 0;
                        game.score = 0;
                        game.speed = 2;
                        game.running = true;
                        spawn_obstacle(&game);
                    }
                }
            }
        }

        game_update(&game);
        view_port_update(view_port);
        furi_delay_ms(50);
    }

    gui_remove_view_port(gui, view_port);
    view_port_free(view_port);
    furi_message_queue_free(event_queue);

    return 0;
}
```

---

# How To Build

Put the folder into:

```text
flipperzero-firmware/applications_user/
```

Then build firmware:

```bash
./fbt fap_bass_runner
```

The compiled `.fap` file will appear in:

```text
build/f7-firmware-D/.extapps/Games/
```

Copy that `.fap` onto your Flipper Zero SD card.

---

# Controls

* OK = Jump/swim upward
* BACK = Exit game

---

# Future Upgrades

Ideas you can add later:

* Multiple obstacle types
* Sound effects
* Water bubbles
* Boss fish
* High score save system
* Different maps
