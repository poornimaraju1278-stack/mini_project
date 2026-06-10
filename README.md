/*
 * 2D ASCII Graphics Editor in C
 * Characters used : '*' and '_'
 * Canvas          : 2D char array (ROWS x COLS)
 * Shapes          : circle, rectangle, line, triangle
 * Operations      : add, delete, modify objects
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>

/* ================================================================
   CANVAS
   ================================================================ */
#define ROWS 24
#define COLS 70

typedef char Canvas[ROWS][COLS];

/* Fill every cell with a space */
void canvas_init(Canvas cv) {
    for (int r = 0; r < ROWS; r++)
        for (int c = 0; c < COLS; c++)
            cv[r][c] = ' ';
}

/* Print the canvas with a simple border */
void canvas_display(const Canvas cv) {
    printf("\n+");
    for (int c = 0; c < COLS; c++) putchar('-');
    printf("+\n");

    for (int r = 0; r < ROWS; r++) {
        putchar('|');
        for (int c = 0; c < COLS; c++)
            putchar(cv[r][c]);
        printf("|\n");
    }

    printf("+");
    for (int c = 0; c < COLS; c++) putchar('-');
    printf("+\n");
}

/* Safe pixel write */
static void put(Canvas cv, int x, int y, char ch) {
    if (x >= 0 && x < COLS && y >= 0 && y < ROWS)
        cv[y][x] = ch;
}

/* ================================================================
   DRAW FUNCTIONS
   ================================================================ */

/* --- Rectangle: hollow outline --- */
void draw_rectangle(Canvas cv, int x, int y, int w, int h, char ch) {
    for (int i = x; i < x + w; i++) {        /* top / bottom */
        put(cv, i, y,         ch);
        put(cv, i, y + h - 1, ch);
    }
    for (int j = y; j < y + h; j++) {        /* left / right */
        put(cv, x,         j, ch);
        put(cv, x + w - 1, j, ch);
    }
}

/* --- Circle: midpoint algorithm, aspect-corrected --- */
void draw_circle(Canvas cv, int cx, int cy, int r, char ch) {
    int x = r, y = 0, err = 1 - r;
    while (x >= y) {
        /* scale y by 0.5 to correct for tall monospace characters */
        int ys = (int)round(y * 0.5);
        int xs = (int)round(x * 0.5);
        put(cv,  cx + x,  cy + ys, ch);  put(cv,  cx - x,  cy + ys, ch);
        put(cv,  cx + x,  cy - ys, ch);  put(cv,  cx - x,  cy - ys, ch);
        put(cv,  cx + y,  cy + xs, ch);  put(cv,  cx - y,  cy + xs, ch);
        put(cv,  cx + y,  cy - xs, ch);  put(cv,  cx - y,  cy - xs, ch);
        y++;
        if (err < 0) {
            err += 2 * y + 1;
        } else {
            x--;
            err += 2 * (y - x) + 1;
        }
    }
}

/* --- Line: Bresenham's algorithm --- */
void draw_line(Canvas cv, int x1, int y1, int x2, int y2, char ch) {
    int dx =  abs(x2 - x1), sx = x1 < x2 ? 1 : -1;
    int dy = -abs(y2 - y1), sy = y1 < y2 ? 1 : -1;
    int err = dx + dy;

    for (;;) {
        put(cv, x1, y1, ch);
        if (x1 == x2 && y1 == y2) break;
        int e2 = 2 * err;
        if (e2 >= dy) { err += dy; x1 += sx; }
        if (e2 <= dx) { err += dx; y1 += sy; }
    }
}

/* --- Triangle: apex centred above base --- */
void draw_triangle(Canvas cv, int x, int y, int w, int h, char ch) {
    int ax = x + w / 2, ay = y;            /* apex */
    int bl = x,         by = y + h - 1;   /* base-left */
    int br = x + w - 1;                   /* base-right (same row) */

    draw_line(cv, ax, ay, bl, by, ch);    /* left edge  */
    draw_line(cv, ax, ay, br, by, ch);    /* right edge */
    draw_line(cv, bl, by, br, by, ch);    /* base       */
}

/* ================================================================
   OBJECT STORE
   ================================================================ */
#define MAX_OBJ 64

typedef enum { T_RECT, T_CIRCLE, T_LINE, T_TRIANGLE } ShapeType;

typedef struct {
    ShapeType type;
    int  x, y;     /* origin or centre */
    int  w, h;     /* width/height, radius, or x2/y2 for line */
    char ch;       /* '*' or '_' */
    int  alive;    /* 1 = active, 0 = deleted */
} Object;

static Object store[MAX_OBJ];
static int    obj_n = 0;           /* total ever created (inc. deleted) */

/* Render every live object onto a fresh canvas and display */
void render(void) {
    Canvas cv;
    canvas_init(cv);

    for (int i = 0; i < obj_n; i++) {
        if (!store[i].alive) continue;
        switch (store[i].type) {
            case T_RECT:
                draw_rectangle(cv, store[i].x, store[i].y,
                                   store[i].w, store[i].h, store[i].ch);
                break;
            case T_CIRCLE:
                draw_circle(cv, store[i].x, store[i].y,
                                store[i].w, store[i].ch);
                break;
            case T_LINE:
                draw_line(cv, store[i].x, store[i].y,
                              store[i].w, store[i].h, store[i].ch);
                break;
            case T_TRIANGLE:
                draw_triangle(cv, store[i].x, store[i].y,
                                  store[i].w, store[i].h, store[i].ch);
                break;
        }
    }

    canvas_display(cv);
}

/* Add a new object; returns its index or -1 on overflow */
int object_add(ShapeType type, int x, int y, int w, int h, char ch) {
    if (obj_n >= MAX_OBJ) { printf("Object limit reached.\n"); return -1; }
    store[obj_n] = (Object){ type, x, y, w, h, ch, 1 };
    printf("  -> Added as object #%d\n", obj_n);
    return obj_n++;
}

/* Delete by index (soft delete) */
void object_delete(int idx) {
    if (idx < 0 || idx >= obj_n || !store[idx].alive) {
        printf("  -> No active object at index %d.\n", idx);
        return;
    }
    store[idx].alive = 0;
    printf("  -> Object #%d deleted.\n", idx);
}

/* Modify any field of an existing object */
void object_modify(int idx, int x, int y, int w, int h, char ch) {
    if (idx < 0 || idx >= obj_n || !store[idx].alive) {
        printf("  -> No active object at index %d.\n", idx);
        return;
    }
    store[idx].x = x;  store[idx].y = y;
    store[idx].w = w;  store[idx].h = h;
    store[idx].ch = ch;
    printf("  -> Object #%d modified.\n", idx);
}

/* Print all active objects */
void object_list(void) {
    const char *names[] = { "rectangle", "circle", "line", "triangle" };
    int found = 0;
    printf("\n  ID  Type        x    y    w/r  h/y2  ch\n");
    printf("  --  ----------  ---  ---  ---  ----  --\n");
    for (int i = 0; i < obj_n; i++) {
        if (!store[i].alive) continue;
        printf("  %-3d %-10s  %-4d %-4d %-4d %-5d '%c'\n",
               i, names[store[i].type],
               store[i].x, store[i].y,
               store[i].w, store[i].h,
               store[i].ch);
        found = 1;
    }
    if (!found) printf("  (no objects)\n");
    putchar('\n');
}

/* ================================================================
   HELPER: validated character input
   ================================================================ */
static char pick_char(void) {
    char ch;
    printf("  Character (* or _): ");
    scanf(" %c", &ch);
    return (ch == '_') ? '_' : '*';
}

/* ================================================================
   MAIN MENU
   ================================================================ */
static void print_menu(void) {
    printf("\n+-------------------------------+\n");
    printf("|  2D ASCII Graphics Editor     |\n");
    printf("+-------------------------------+\n");
    printf("| 1) Add Rectangle              |\n");
    printf("| 2) Add Circle                 |\n");
    printf("| 3) Add Line                   |\n");
    printf("| 4) Add Triangle               |\n");
    printf("| 5) Delete Object              |\n");
    printf("| 6) Modify Object              |\n");
    printf("| 7) List Objects               |\n");
    printf("| 8) Render / Display Canvas    |\n");
    printf("| 0) Quit                       |\n");
    printf("+-------------------------------+\n");
    printf("Choice: ");
}

int main(void) {
    int choice;
    int x, y, w, h, idx;
    char ch;

    /* Demo scene so the canvas is not empty at start */
    printf("\n=== 2D ASCII Graphics Editor ===\n");
    printf("Loading demo scene...\n");
    object_add(T_RECT,     2,  1, 20,  8, '*');
    object_add(T_CIRCLE,  50,  8,  7,  7, '*');
    object_add(T_LINE,     0,  0, 69, 23, '_');
    object_add(T_TRIANGLE,28,  2, 16,  9, '_');
    render();

    for (;;) {
        print_menu();
        if (scanf("%d", &choice) != 1) break;

        switch (choice) {

        /* ---- ADD SHAPES ---- */
        case 1:
            printf("  Top-left x y : "); scanf("%d %d", &x, &y);
            printf("  Width height : "); scanf("%d %d", &w, &h);
            ch = pick_char();
            object_add(T_RECT, x, y, w, h, ch);
            render();
            break;

        case 2:
            printf("  Centre x y  : "); scanf("%d %d", &x, &y);
            printf("  Radius      : "); scanf("%d", &w);
            ch = pick_char();
            object_add(T_CIRCLE, x, y, w, w, ch);
            render();
            break;

        case 3:
            printf("  Start x1 y1 : "); scanf("%d %d", &x, &y);
            printf("  End   x2 y2 : "); scanf("%d %d", &w, &h);
            ch = pick_char();
            object_add(T_LINE, x, y, w, h, ch);
            render();
            break;

        case 4:
            printf("  Top-left x y : "); scanf("%d %d", &x, &y);
            printf("  Width height : "); scanf("%d %d", &w, &h);
            ch = pick_char();
            object_add(T_TRIANGLE, x, y, w, h, ch);
            render();
            break;

        /* ---- DELETE ---- */
        case 5:
            object_list();
            printf("  Object index to delete: "); scanf("%d", &idx);
            object_delete(idx);
            render();
            break;

        /* ---- MODIFY ---- */
        case 6:
            object_list();
            printf("  Object index to modify : "); scanf("%d", &idx);
            if (idx < 0 || idx >= obj_n || !store[idx].alive) {
                printf("  -> Invalid index.\n");
                break;
            }
            if (store[idx].type == T_CIRCLE) {
                printf("  New centre x y : "); scanf("%d %d", &x, &y);
                printf("  New radius     : "); scanf("%d", &w);
                h = w;
            } else if (store[idx].type == T_LINE) {
                printf("  New x1 y1 : "); scanf("%d %d", &x, &y);
                printf("  New x2 y2 : "); scanf("%d %d", &w, &h);
            } else {
                printf("  New top-left x y : "); scanf("%d %d", &x, &y);
                printf("  New width height : "); scanf("%d %d", &w, &h);
            }
            ch = pick_char();
            object_modify(idx, x, y, w, h, ch);
            render();
            break;

        /* ---- UTILITIES ---- */
        case 7:
            object_list();
            break;

        case 8:
            render();
            break;

        case 0:
            printf("Goodbye!\n");
            return 0;

        default:
            printf("  Unknown option. Try again.\n");
        }
    }

    return 0;
}
