# IA
Tecnicas de IA - Patrulha

#include "SDL.h"

const int WIDTH = 800;
const int HEIGHT = 600;
const int FRAME_RATE = 60;
const int SPEED = 10;
const int SPEED_NPC = 8;
const int WALK = 150;
const int DISTANCE = 150;

void init();
void input();
void npc();
void draw();
void update();
void close();
Uint32 wait(Uint32 t0);
SDL_Surface *changeState();

enum Direction
{
	UP,
	DOWN,
	LEFT,
	RIGHT
};

enum StateMachine
{
	PATROL,
	PURSUIT
};

SDL_Surface *screen, *player, *enemy01, *red_box, *green_box;
SDL_Rect src_player, dest_player, src_enemy01, dest_enemy01;
bool quit;
int x, y;
Direction walk_dir;
StateMachine state_npc;

int main(int argc, char *argv[]) 
{
    // Inicializa a SDL
	init();

    // Variaveis
    Uint32 t0 = SDL_GetTicks();

    // Main Loop
    while (!quit)
    {
        // Input
        input();
        npc();

        // Draw
        draw();

        // Update
        update();

        // Fixed Rate
        t0 = wait(t0);
    }

    // Finaliza a SDL
    close();

    // Sai do programa
    exit(0);
}

void init()
{
    if (SDL_Init(SDL_INIT_VIDEO) == -1)
    {
		exit(-1);
    }

	screen = SDL_SetVideoMode(WIDTH, HEIGHT, 32, SDL_HWSURFACE|SDL_DOUBLEBUF);

	if (screen == NULL) 
    {
		SDL_Quit();
		exit(-1);
	}

    player = SDL_LoadBMP("blue.bmp");
    red_box = SDL_LoadBMP("red.bmp");
	green_box = SDL_LoadBMP("green.bmp");

    if ((player == NULL) || (red_box == NULL) || (green_box == NULL))
    {
        SDL_Quit();
		exit(-1);
    }

	enemy01 = green_box;

    x = 0;
    y = 0;

    src_player.x = 0;
    src_player.y = 0;
    src_player.w = player->w;
    src_player.h = player->h;

    dest_player.x = (screen->w / 2) - player->w;
    dest_player.y = (screen->h / 2) - player->h;
    dest_player.w = player->w;
    dest_player.h = player->h; 

    src_enemy01.x = 0;
    src_enemy01.y = 0;
    src_enemy01.w = enemy01->w;
    src_enemy01.h = enemy01->h;

    dest_enemy01.x = WALK;
    dest_enemy01.y = WALK;
    dest_enemy01.w = enemy01->w;
    dest_enemy01.h = enemy01->h; 

	walk_dir = RIGHT;
	state_npc = PATROL;
}

void input()
{
    SDL_Event e;
    while(SDL_PollEvent(&e)) 
    {
        if (e.type == SDL_KEYDOWN)
        {
            switch(e.key.keysym.sym)
            {
                case SDLK_ESCAPE:
                    quit = true;
                    break;
                case SDLK_UP:
                    y -= SPEED;
                    break;
                case SDLK_DOWN:
                    y += SPEED;
                    break;
                case SDLK_LEFT:
                    x -= SPEED;
                    break;
                case SDLK_RIGHT:
                    x += SPEED;
                    break;
            }
        }
        else if (e.type == SDL_KEYUP) 
        {
            switch(e.key.keysym.sym)
            {
                case SDLK_ESCAPE:
                    quit = true;
                    break;
                case SDLK_UP:
                    y = 0;
                    break;
                case SDLK_DOWN:
                    y = 0;
                    break;
                case SDLK_LEFT:
                    x = 0;
                    break;
                case SDLK_RIGHT:
                    x = 0;
                    break;
            }
        }
    }

    if ((x != 0) && ((dest_player.x + x) > 0) && ((dest_player.x + x) < (screen->w - player->w)))
    {
        dest_player.x += x;
    }
    if ((y != 0) && ((dest_player.y + y) > 0) && ((dest_player.y + y) < (screen->h - player->h)))
    {
        dest_player.y += y;
    }
}

void npc()
{
	if (((dest_player.x - dest_enemy01.x) < DISTANCE) && ((dest_player.y - dest_enemy01.y) < DISTANCE))
	{
		state_npc = PURSUIT;
	}
	else
	{
		state_npc = PATROL;
	}

    //TODO: O movimento deve ser circular em qualquer lugar
	if (state_npc == PATROL)
	{
		if (walk_dir == RIGHT)
		{		
			if (dest_enemy01.x + SPEED_NPC > WALK)
			{
				walk_dir = DOWN;
			}
			else
			{
				dest_enemy01.x += SPEED_NPC;
			}
		}
		else if (walk_dir == DOWN)
		{
			if (dest_enemy01.y + SPEED_NPC > WALK)
			{
				walk_dir = LEFT;
			}
			else
			{
				dest_enemy01.y += SPEED_NPC;
			}
		}
		else if (walk_dir == LEFT)
		{
			if (dest_enemy01.x - SPEED_NPC <= 0)
			{
				walk_dir = UP;
			}
			else
			{
				dest_enemy01.x -= SPEED_NPC;
			}
		}
		else if (walk_dir == UP)
		{
			if (dest_enemy01.y - SPEED_NPC <= 0)
			{
				walk_dir = RIGHT;
			}
			else
			{
				dest_enemy01.y -= SPEED_NPC;
			}
		}
	}
	else if (state_npc == PURSUIT)
	{
		if (dest_enemy01.x > (dest_player.x + 4))
		{
			dest_enemy01.x -= SPEED_NPC;
		}
		else if (dest_enemy01.x < (dest_player.x - 4))
		{
			dest_enemy01.x += SPEED_NPC;
		}

		if (dest_enemy01.y > (dest_player.y + 4))
		{
			dest_enemy01.y -= SPEED_NPC;
		}
		else if (dest_enemy01.y < (dest_player.y - 4))
		{
			dest_enemy01.y += SPEED_NPC;
		}
	}   
}

void draw()
{
    SDL_FillRect(screen, NULL, 0);
    SDL_BlitSurface(player, &src_player, screen, &dest_player);
	enemy01 = changeState();
    SDL_BlitSurface(enemy01, &src_enemy01, screen, &dest_enemy01);
}

void update()
{
    SDL_Flip(screen);
}

void close()
{
    SDL_FreeSurface(screen);
    SDL_FreeSurface(player);
    SDL_FreeSurface(enemy01);
    SDL_Quit();
}

Uint32 wait(Uint32 t0)
{
	Uint32 t1 = SDL_GetTicks();

	if ((t1 - t0) < FRAME_RATE)
		SDL_Delay(FRAME_RATE - (t1 - t0));

    return t1;
}

SDL_Surface *changeState()
{
	if (state_npc == PATROL)
	{
		return green_box;
	}
	else if (state_npc == PURSUIT)
	{
		return red_box;
	}
	else 
	{
		return NULL;
	}
}
