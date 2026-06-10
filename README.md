#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <string.h>

/* Canvas dimensions */
#define CANVAS_WIDTH 50
#define CANVAS_HEIGHT 20

/* Maximum number of objects that can be stored */
#define MAX_OBJECTS 50

/* Structure to store object information */
typedef struct {
    int id;                    /* Unique object identifier */
    char shape_type;           /* 'L' for Line, 'R' for Rectangle, 'T' for Triangle, 'C' for Circle */
    int x1, y1;               /* Starting point or center */
    int x2, y2;               /* Ending point or second corner */
    int x3, y3;               /* Third point (for triangle) */
    int radius;               /* Radius (for circle) */
    int is_active;            /* 1 if active, 0 if deleted */
} GraphicsObject;

/* Global canvas array */
char canvas[CANVAS_HEIGHT][CANVAS_WIDTH];

/* Array to store all objects */
GraphicsObject objects[MAX_OBJECTS];
int object_count = 0;
int next_id = 1;

/* ========== UTILITY FUNCTIONS ========== */

/*
 * Function: initializeCanvas()
 * Description: Initializes the canvas with '_' characters
 * Parameters: None
 * Return: void
 */
void initializeCanvas() {
    int i, j;
    for (i = 0; i < CANVAS_HEIGHT; i++) {
        for (j = 0; j < CANVAS_WIDTH; j++) {
            canvas[i][j] = '_';
        }
    }
}

/*
 * Function: displayCanvas()
 * Description: Displays the current canvas with borders
 * Parameters: None
 * Return: void
 */
void displayCanvas() {
    int i, j;
    printf("\n");
    printf("+-");
    for (j = 0; j < CANVAS_WIDTH; j++) {
        printf("-");
    }
    printf("-+\n");
    
    for (i = 0; i < CANVAS_HEIGHT; i++) {
        printf("| ");
        for (j = 0; j < CANVAS_WIDTH; j++) {
            printf("%c", canvas[i][j]);
        }
        printf(" |\n");
    }
    
    printf("+-");
    for (j = 0; j < CANVAS_WIDTH; j++) {
        printf("-");
    }
    printf("-+\n\n");
}

/*
 * Function: isWithinBounds()
 * Description: Checks if coordinates are within canvas boundaries
 * Parameters: x, y - coordinates to check
 * Return: 1 if within bounds, 0 otherwise
 */
int isWithinBounds(int x, int y) {
    return (x >= 0 && x < CANVAS_WIDTH && y >= 0 && y < CANVAS_HEIGHT);
}

/*
 * Function: clearCanvas()
 * Description: Clears all active objects from the canvas and reinitializes it
 * Parameters: None
 * Return: void
 */
void clearCanvas() {
    initializeCanvas();
}

/*
 * Function: redrawCanvas()
 * Description: Redraws the canvas based on all active objects
 * Parameters: None
 * Return: void
 */
void redrawCanvas() {
    int i, j;
    
    /* Clear canvas */
    clearCanvas();
    
    /* Redraw all active objects */
    for (i = 0; i < object_count; i++) {
        if (objects[i].is_active) {
            if (objects[i].shape_type == 'L') {
                drawLine(objects[i].x1, objects[i].y1, objects[i].x2, objects[i].y2);
            } else if (objects[i].shape_type == 'R') {
                drawRectangle(objects[i].x1, objects[i].y1, objects[i].x2, objects[i].y2);
            } else if (objects[i].shape_type == 'T') {
                drawTriangle(objects[i].x1, objects[i].y1, objects[i].x2, objects[i].y2, objects[i].x3, objects[i].y3);
            } else if (objects[i].shape_type == 'C') {
                drawCircle(objects[i].x1, objects[i].y1, objects[i].radius);
            }
        }
    }
}

/* ========== DRAWING FUNCTIONS ========== */

/*
 * Function: drawLine()
 * Description: Draws a line from (x1,y1) to (x2,y2) using Bresenham's line algorithm
 * Parameters: x1, y1 - starting point; x2, y2 - ending point
 * Return: void
 */
void drawLine(int x1, int y1, int x2, int y2) {
    int dx = abs(x2 - x1);
    int dy = abs(y2 - y1);
    int sx = (x1 < x2) ? 1 : -1;
    int sy = (y1 < y2) ? 1 : -1;
    int err = dx - dy;
    int x = x1, y = y1;
    
    while (1) {
        if (isWithinBounds(x, y)) {
            canvas[y][x] = '*';
        }
        
        if (x == x2 && y == y2) break;
        
        int e2 = 2 * err;
        if (e2 > -dy) {
            err -= dy;
            x += sx;
        }
        if (e2 < dx) {
            err += dx;
            y += sy;
        }
    }
}

/*
 * Function: drawRectangle()
 * Description: Draws a rectangle with corners at (x1,y1) and (x2,y2)
 * Parameters: x1, y1 - top-left corner; x2, y2 - bottom-right corner
 * Return: void
 */
void drawRectangle(int x1, int y1, int x2, int y2) {
    int minX = (x1 < x2) ? x1 : x2;
    int maxX = (x1 > x2) ? x1 : x2;
    int minY = (y1 < y2) ? y1 : y2;
    int maxY = (y1 > y2) ? y1 : y2;
    
    int i, j;
    
    /* Draw top and bottom edges */
    for (i = minX; i <= maxX; i++) {
        if (isWithinBounds(i, minY)) canvas[minY][i] = '*';
        if (isWithinBounds(i, maxY)) canvas[maxY][i] = '*';
    }
    
    /* Draw left and right edges */
    for (i = minY; i <= maxY; i++) {
        if (isWithinBounds(minX, i)) canvas[i][minX] = '*';
        if (isWithinBounds(maxX, i)) canvas[i][maxX] = '*';
    }
}

/*
 * Function: drawTriangle()
 * Description: Draws a triangle with vertices at (x1,y1), (x2,y2), (x3,y3)
 * Parameters: x1, y1, x2, y2, x3, y3 - coordinates of three vertices
 * Return: void
 */
void drawTriangle(int x1, int y1, int x2, int y2, int x3, int y3) {
    /* Draw three lines connecting the vertices */
    drawLine(x1, y1, x2, y2);
    drawLine(x2, y2, x3, y3);
    drawLine(x3, y3, x1, y1);
}

/*
 * Function: drawCircle()
 * Description: Draws a circle with center at (cx,cy) and given radius
 *              Uses Midpoint Circle Algorithm
 * Parameters: cx, cy - center coordinates; radius - radius of circle
 * Return: void
 */
void drawCircle(int cx, int cy, int radius) {
    int x = 0;
    int y = radius;
    int d = 3 - 2 * radius;
    
    while (x <= y) {
        /* Plot points in all 8 octants */
        if (isWithinBounds(cx + x, cy + y)) canvas[cy + y][cx + x] = '*';
        if (isWithinBounds(cx - x, cy + y)) canvas[cy + y][cx - x] = '*';
        if (isWithinBounds(cx + x, cy - y)) canvas[cy - y][cx + x] = '*';
        if (isWithinBounds(cx - x, cy - y)) canvas[cy - y][cx - x] = '*';
        if (isWithinBounds(cx + y, cy + x)) canvas[cy + x][cx + y] = '*';
        if (isWithinBounds(cx - y, cy + x)) canvas[cy + x][cx - y] = '*';
        if (isWithinBounds(cx + y, cy - x)) canvas[cy - x][cx + y] = '*';
        if (isWithinBounds(cx - y, cy - x)) canvas[cy - x][cx - y] = '*';
        
        if (d < 0) {
            d = d + 4 * x + 6;
        } else {
            d = d + 4 * (x - y) + 10;
            y--;
        }
        x++;
    }
}

/* ========== OBJECT MANAGEMENT FUNCTIONS ========== */

/*
 * Function: addObject()
 * Description: Adds a new graphics object to the array and draws it
 * Parameters: shape_type - type of shape; x1, y1, x2, y2, x3, y3, radius - shape parameters
 * Return: void
 */
void addObject(char shape_type, int x1, int y1, int x2, int y2, int x3, int y3, int radius) {
    if (object_count >= MAX_OBJECTS) {
        printf("\nError: Maximum objects reached!\n");
        return;
    }
    
    objects[object_count].id = next_id++;
    objects[object_count].shape_type = shape_type;
    objects[object_count].x1 = x1;
    objects[object_count].y1 = y1;
    objects[object_count].x2 = x2;
    objects[object_count].y2 = y2;
    objects[object_count].x3 = x3;
    objects[object_count].y3 = y3;
    objects[object_count].radius = radius;
    objects[object_count].is_active = 1;
    
    printf("\nObject %d added successfully!\n", objects[object_count].id);
    object_count++;
    
    redrawCanvas();
}

/*
 * Function: deleteObject()
 * Description: Deletes an object by marking it as inactive and clearing it from canvas
 * Parameters: object_id - ID of object to delete
 * Return: void
 */
void deleteObject(int object_id) {
    int i;
    
    for (i = 0; i < object_count; i++) {
        if (objects[i].id == object_id && objects[i].is_active) {
            objects[i].is_active = 0;
            printf("\nObject %d deleted successfully!\n", object_id);
            redrawCanvas();
            return;
        }
    }
    
    printf("\nError: Object with ID %d not found!\n", object_id);
}

/*
 * Function: modifyObject()
 * Description: Modifies position or size of an existing object
 * Parameters: object_id - ID of object to modify; new parameters
 * Return: void
 */
void modifyObject(int object_id, int x1, int y1, int x2, int y2, int x3, int y3, int radius) {
    int i;
    
    for (i = 0; i < object_count; i++) {
        if (objects[i].id == object_id && objects[i].is_active) {
            objects[i].x1 = x1;
            objects[i].y1 = y1;
            objects[i].x2 = x2;
            objects[i].y2 = y2;
            objects[i].x3 = x3;
            objects[i].y3 = y3;
            objects[i].radius = radius;
            
            printf("\nObject %d modified successfully!\n", object_id);
            redrawCanvas();
            return;
        }
    }
    
    printf("\nError: Object with ID %d not found!\n", object_id);
}

/*
 * Function: listObjects()
 * Description: Lists all active objects with their properties
 * Parameters: None
 * Return: void
 */
void listObjects() {
    int i;
    int found = 0;
    
    printf("\n========== ACTIVE OBJECTS ==========\n");
    for (i = 0; i < object_count; i++) {
        if (objects[i].is_active) {
            found = 1;
            printf("\nObject ID: %d\n", objects[i].id);
            printf("Shape Type: ");
            
            if (objects[i].shape_type == 'L') {
                printf("LINE\n");
                printf("From (%d, %d) to (%d, %d)\n", objects[i].x1, objects[i].y1, objects[i].x2, objects[i].y2);
            } else if (objects[i].shape_type == 'R') {
                printf("RECTANGLE\n");
                printf("Corner 1: (%d, %d), Corner 2: (%d, %d)\n", objects[i].x1, objects[i].y1, objects[i].x2, objects[i].y2);
            } else if (objects[i].shape_type == 'T') {
                printf("TRIANGLE\n");
                printf("Vertex 1: (%d, %d), Vertex 2: (%d, %d), Vertex 3: (%d, %d)\n", 
                       objects[i].x1, objects[i].y1, objects[i].x2, objects[i].y2, objects[i].x3, objects[i].y3);
            } else if (objects[i].shape_type == 'C') {
                printf("CIRCLE\n");
                printf("Center: (%d, %d), Radius: %d\n", objects[i].x1, objects[i].y1, objects[i].radius);
            }
        }
    }
    
    if (!found) {
        printf("\nNo active objects found!\n");
    }
    printf("====================================\n");
}

/* ========== MENU AND INPUT FUNCTIONS ========== */

/*
 * Function: drawLineMenu()
 * Description: Menu for drawing a line
 * Parameters: None
 * Return: void
 */
void drawLineMenu() {
    int x1, y1, x2, y2;
    
    printf("\n========== DRAW LINE ==========\n");
    printf("Enter starting point (x1 y1): ");
    scanf("%d %d", &x1, &y1);
    printf("Enter ending point (x2 y2): ");
    scanf("%d %d", &x2, &y2);
    
    if (!isWithinBounds(x1, y1) || !isWithinBounds(x2, y2)) {
        printf("Error: Coordinates out of bounds! (0-%d, 0-%d)\n", CANVAS_WIDTH-1, CANVAS_HEIGHT-1);
        return;
    }
    
    addObject('L', x1, y1, x2, y2, 0, 0, 0);
}

/*
 * Function: drawRectangleMenu()
 * Description: Menu for drawing a rectangle
 * Parameters: None
 * Return: void
 */
void drawRectangleMenu() {
    int x1, y1, x2, y2;
    
    printf("\n========== DRAW RECTANGLE ==========\n");
    printf("Enter top-left corner (x1 y1): ");
    scanf("%d %d", &x1, &y1);
    printf("Enter bottom-right corner (x2 y2): ");
    scanf("%d %d", &x2, &y2);
    
    if (!isWithinBounds(x1, y1) || !isWithinBounds(x2, y2)) {
        printf("Error: Coordinates out of bounds! (0-%d, 0-%d)\n", CANVAS_WIDTH-1, CANVAS_HEIGHT-1);
        return;
    }
    
    addObject('R', x1, y1, x2, y2, 0, 0, 0);
}

/*
 * Function: drawTriangleMenu()
 * Description: Menu for drawing a triangle
 * Parameters: None
 * Return: void
 */
void drawTriangleMenu() {
    int x1, y1, x2, y2, x3, y3;
    
    printf("\n========== DRAW TRIANGLE ==========\n");
    printf("Enter vertex 1 (x1 y1): ");
    scanf("%d %d", &x1, &y1);
    printf("Enter vertex 2 (x2 y2): ");
    scanf("%d %d", &x2, &y2);
    printf("Enter vertex 3 (x3 y3): ");
    scanf("%d %d", &x3, &y3);
    
    if (!isWithinBounds(x1, y1) || !isWithinBounds(x2, y2) || !isWithinBounds(x3, y3)) {
        printf("Error: Coordinates out of bounds! (0-%d, 0-%d)\n", CANVAS_WIDTH-1, CANVAS_HEIGHT-1);
        return;
    }
    
    addObject('T', x1, y1, x2, y2, x3, y3, 0);
}

/*
 * Function: drawCircleMenu()
 * Description: Menu for drawing a circle
 * Parameters: None
 * Return: void
 */
void drawCircleMenu() {
    int cx, cy, radius;
    
    printf("\n========== DRAW CIRCLE ==========\n");
    printf("Enter center (cx cy): ");
    scanf("%d %d", &cx, &cy);
    printf("Enter radius: ");
    scanf("%d", &radius);
    
    if (!isWithinBounds(cx, cy)) {
        printf("Error: Center out of bounds! (0-%d, 0-%d)\n", CANVAS_WIDTH-1, CANVAS_HEIGHT-1);
        return;
    }
    
    if (radius < 1) {
        printf("Error: Radius must be at least 1!\n");
        return;
    }
    
    addObject('C', cx, cy, 0, 0, 0, 0, radius);
}

/*
 * Function: deleteObjectMenu()
 * Description: Menu for deleting an object
 * Parameters: None
 * Return: void
 */
void deleteObjectMenu() {
    int object_id;
    
    printf("\n========== DELETE OBJECT ==========\n");
    listObjects();
    printf("\nEnter object ID to delete: ");
    scanf("%d", &object_id);
    
    deleteObject(object_id);
}

/*
 * Function: modifyObjectMenu()
 * Description: Menu for modifying an object
 * Parameters: None
 * Return: void
 */
void modifyObjectMenu() {
    int object_id, i, found = 0;
    
    printf("\n========== MODIFY OBJECT ==========\n");
    listObjects();
    printf("\nEnter object ID to modify: ");
    scanf("%d", &object_id);
    
    /* Find the object */
    for (i = 0; i < object_count; i++) {
        if (objects[i].id == object_id && objects[i].is_active) {
            found = 1;
            break;
        }
    }
    
    if (!found) {
        printf("Error: Object with ID %d not found!\n", object_id);
        return;
    }
    
    /* Get new parameters based on shape type */
    if (objects[i].shape_type == 'L') {
        int x1, y1, x2, y2;
        printf("Enter new starting point (x1 y1): ");
        scanf("%d %d", &x1, &y1);
        printf("Enter new ending point (x2 y2): ");
        scanf("%d %d", &x2, &y2);
        
        if (!isWithinBounds(x1, y1) || !isWithinBounds(x2, y2)) {
            printf("Error: Coordinates out of bounds!\n");
            return;
        }
        
        modifyObject(object_id, x1, y1, x2, y2, 0, 0, 0);
    } else if (objects[i].shape_type == 'R') {
        int x1, y1, x2, y2;
        printf("Enter new top-left corner (x1 y1): ");
        scanf("%d %d", &x1, &y1);
        printf("Enter new bottom-right corner (x2 y2): ");
        scanf("%d %d", &x2, &y2);
        
        if (!isWithinBounds(x1, y1) || !isWithinBounds(x2, y2)) {
            printf("Error: Coordinates out of bounds!\n");
            return;
        }
        
        modifyObject(object_id, x1, y1, x2, y2, 0, 0, 0);
    } else if (objects[i].shape_type == 'T') {
        int x1, y1, x2, y2, x3, y3;
        printf("Enter new vertex 1 (x1 y1): ");
        scanf("%d %d", &x1, &y1);
        printf("Enter new vertex 2 (x2 y2): ");
        scanf("%d %d", &x2, &y2);
        printf("Enter new vertex 3 (x3 y3): ");
        scanf("%d %d", &x3, &y3);
        
        if (!isWithinBounds(x1, y1) || !isWithinBounds(x2, y2) || !isWithinBounds(x3, y3)) {
            printf("Error: Coordinates out of bounds!\n");
            return;
        }
        
        modifyObject(object_id, x1, y1, x2, y2, x3, y3, 0);
    } else if (objects[i].shape_type == 'C') {
        int cx, cy, radius;
        printf("Enter new center (cx cy): ");
        scanf("%d %d", &cx, &cy);
        printf("Enter new radius: ");
        scanf("%d", &radius);
        
        if (!isWithinBounds(cx, cy)) {
            printf("Error: Center out of bounds!\n");
            return;
        }
        
        if (radius < 1) {
            printf("Error: Radius must be at least 1!\n");
            return;
        }
        
        modifyObject(object_id, cx, cy, 0, 0, 0, 0, radius);
    }
}

/*
 * Function: displayMenu()
 * Description: Displays the main menu
 * Parameters: None
 * Return: void
 */
void displayMenu() {
    printf("\n╔════════════════════════════════════════════╗\n");
    printf("║     2D GRAPHICS EDITOR - MAIN MENU         ║\n");
    printf("╠════════════════════════════════════════════╣\n");
    printf("║  1. Draw Line                              ║\n");
    printf("║  2. Draw Rectangle                         ║\n");
    printf("║  3. Draw Triangle                          ║\n");
    printf("║  4. Draw Circle                            ║\n");
    printf("║  5. Display Canvas                         ║\n");
    printf("║  6. List All Objects                       ║\n");
    printf("║  7. Modify Object                          ║\n");
    printf("║  8. Delete Object                          ║\n");
    printf("║  9. Clear Canvas                           ║\n");
    printf("║  0. Exit                                   ║\n");
    printf("╚════════════════════════════════════════════╝\n");
    printf("Enter your choice: ");
}

/*
 * Function: main()
 * Description: Main program - handles menu and user input
 * Parameters: None
 * Return: 0
 */
int main() {
    int choice;
    int running = 1;
    
    printf("\n╔════════════════════════════════════════════╗\n");
    printf("║    WELCOME TO 2D GRAPHICS EDITOR!          ║\n");
    printf("║                                            ║\n");
    printf("║  Canvas Size: %d x %d                     ║\n", CANVAS_WIDTH, CANVAS_HEIGHT);
    printf("║  Maximum Objects: %d                      ║\n", MAX_OBJECTS);
    printf("╚════════════════════════════════════════════╝\n", CANVAS_WIDTH, CANVAS_HEIGHT);
    
    /* Initialize canvas */
    initializeCanvas();
    displayCanvas();
    
    while (running) {
        displayMenu();
        scanf("%d", &choice);
        
        switch (choice) {
            case 1:
                drawLineMenu();
                displayCanvas();
                break;
                
            case 2:
                drawRectangleMenu();
                displayCanvas();
                break;
                
            case 3:
                drawTriangleMenu();
                displayCanvas();
                break;
                
            case 4:
                drawCircleMenu();
                displayCanvas();
                break;
                
            case 5:
                printf("\n========== CURRENT CANVAS ==========\n");
                displayCanvas();
                break;
                
            case 6:
                listObjects();
                break;
                
            case 7:
                modifyObjectMenu();
                displayCanvas();
                break;
                
            case 8:
                deleteObjectMenu();
                displayCanvas();
                break;
                
            case 9:
                initializeCanvas();
                object_count = 0;
                next_id = 1;
                printf("\nCanvas cleared!\n");
                displayCanvas();
                break;
                
            case 0:
                printf("\nThank you for using 2D Graphics Editor!\n");
                printf("Goodbye!\n\n");
                running = 0;
                break;
                
            default:
                printf("Invalid choice! Please try again.\n");
        }
    }
    
    return 0;
}

/*
==========================================
SAMPLE INPUT AND OUTPUT:

Welcome to 2D Graphics Editor!

Canvas Size: 50 x 20
Maximum Objects: 50

+-----------...----------+
|_____________________...|
|_____________________...|
... (20 rows)
+-----------...----------+

========== 2D GRAPHICS EDITOR - MAIN MENU ==========
1. Draw Line
2. Draw Rectangle
3. Draw Triangle
4. Draw Circle
5. Display Canvas
6. List All Objects
7. Modify Object
8. Delete Object
9. Clear Canvas
0. Exit

Sample Session:

> 1
========== DRAW LINE ==========
Enter starting point (x1 y1): 5 5
Enter ending point (x2 y2): 15 10

Object 1 added successfully!

+---...---+
|____*____|
|____*____|
|___*_____|
|__*______|
|_*_______|
|*________|
... (rest of canvas)
+---...---+

> 2
========== DRAW RECTANGLE ==========
Enter top-left corner (x1 y1): 20 3
Enter bottom-right corner (x2 y2): 30 8

Object 2 added successfully!

[Canvas now shows both line and rectangle]

> 6
========== ACTIVE OBJECTS ==========

Object ID: 1
Shape Type: LINE
From (5, 5) to (15, 10)

Object ID: 2
Shape Type: RECTANGLE
Corner 1: (20, 3), Corner 2: (30, 8)

====================================

> 8
========== DELETE OBJECT ==========
========== ACTIVE OBJECTS ==========
... (shows list)

Enter object ID to delete: 1

Object 1 deleted successfully!

[Canvas updates and line is removed]

> 0

Thank you for using 2D Graphics Editor!
Goodbye!

==========================================

Compilation and Execution Instructions:

For GCC:
$ gcc graphics_editor.c -o graphics_editor -lm
$ ./graphics_editor

For Turbo C:
1. Copy the code into a .c file
2. Compile: Alt+F9
3. Run: Ctrl+F9

Note: The -lm flag is for linking the math library (used in circle drawing)

==========================================
*/
