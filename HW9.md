# HW9 - A* Search
## Problem 1:

In the assignment, the blank title is labeled with number 0, and 
other titles are labeled with non-zero numbers from 1~15. 
The goal locations are as shown below, and the initial locations 
are provided through program arguments by listing tile indexes in a row-major order. 

For example, the command
`./puzzle 2 3 0 4 1 6 7 8 5 9 10 12 13 14 11 15`
is to solve a 15-puzzle problem, in which the tiles are initially placed as follows, and are to be
moved to their goal locations shown above:

- Solution code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define N 4
#define NxN (N*N)
#define TRUE 1
#define FALSE 0

struct node {
	int tiles[N][N];
	int f, g, h;
	short zero_row, zero_column;	/* location (row and colum) of blank tile 0 */
	struct node *next;
	struct node *parent;			/* used to trace back the solution */
};

int goal_row_for_value[NxN];
int goal_col_for_value[NxN];
struct node *start_state, *goal_state;
struct node *open_list = NULL, *closed_list = NULL;
struct node *successor_nodes[4];

void print_a_node(struct node *current_node) {
	int row, col;

	for (row = 0; row < N; row++) {
		for (col = 0; col < N; col++)
			printf("%2d ", current_node->tiles[row][col]);
		printf("\n");
	}
	printf("\n");
}

struct node *initialize(char **argv) {
	int i, row, col, arg_index, tile_value;
	struct node *temp_node;

	temp_node = (struct node *) malloc(sizeof(struct node));
	arg_index = 1;

	for (row = 0; row < N; row++)
		for (col = 0; col < N; col++) {
			tile_value = atoi(argv[arg_index++]);
			temp_node->tiles[row][col] = tile_value;
			if (tile_value == 0) {
				temp_node->zero_row = row;
				temp_node->zero_column = col;
			}
		}

	temp_node->f = 0;
	temp_node->g = 0;
	temp_node->h = 0;
	temp_node->next = NULL;
	temp_node->parent = NULL;

	start_state = temp_node;

	printf("initial state\n");
	print_a_node(start_state);

	temp_node = (struct node *) malloc(sizeof(struct node));
	goal_row_for_value[0] = 3;
	goal_col_for_value[0] = 3;

	for (arg_index = 1; arg_index < NxN; arg_index++) {
		row = (arg_index - 1) / N;
		col = (arg_index - 1) % N;

		goal_row_for_value[arg_index] = row;
		goal_col_for_value[arg_index] = col;
		temp_node->tiles[row][col] = arg_index;
	}

	temp_node->tiles[N - 1][N - 1] = 0;
	temp_node->f = 0;
	temp_node->g = 0;
	temp_node->h = 0;
	temp_node->next = NULL;

	goal_state = temp_node;

	printf("goal state\n");
	print_a_node(goal_state);

	return start_state;
}

void merge_to_open() { 
	for (int idx = 0; idx < 4; idx++) {
		struct node *new_node = successor_nodes[idx];
		if (new_node == NULL)
			continue;

		struct node **insert_ptr = &open_list;

		while (*insert_ptr != NULL && (*insert_ptr)->f <= new_node->f) {
			insert_ptr = &((*insert_ptr)->next);
		}

		new_node->next = *insert_ptr;
		*insert_ptr = new_node;
	}
}

void swap(int row1, int column1, int row2, int column2, struct node *puzzle_node) {
	int temp_tile = puzzle_node->tiles[row1][column1];
	puzzle_node->tiles[row1][column1] = puzzle_node->tiles[row2][column2]; 
	puzzle_node->tiles[row2][column2] = temp_tile; 
}

void update_fgh(int index) {
	struct node *child_node = successor_nodes[index];

	if (child_node == NULL)
		return; 

	child_node->g = child_node->parent->g + 1;

	int total_dist = 0;

	for (int row = 0; row < N; row++) {
		for (int col = 0; col < N; col++) {
			int tile_value = child_node->tiles[row][col];

			if (tile_value != 0) {
				int target_row = goal_row_for_value[tile_value];
				int target_col = goal_col_for_value[tile_value];

				total_dist += abs(row - target_row) + abs(col - target_col);
			}
		}
	}

	child_node->h = total_dist;
	child_node->f = child_node->g + child_node->h;
}

void move_down(struct node *current_node) {
	swap(current_node->zero_row,
	     current_node->zero_column,
	     current_node->zero_row + 1,
	     current_node->zero_column,
	     current_node);
	current_node->zero_row++;
}

void move_right(struct node *current_node) {
	swap(current_node->zero_row,
	     current_node->zero_column,
	     current_node->zero_row,
	     current_node->zero_column + 1,
	     current_node);
	current_node->zero_column++;
}

void move_up(struct node *current_node) {
	swap(current_node->zero_row,
	     current_node->zero_column,
	     current_node->zero_row - 1,
	     current_node->zero_column,
	     current_node);
	current_node->zero_row--;
}

void move_left(struct node *current_node) {
	swap(current_node->zero_row,
	     current_node->zero_column,
	     current_node->zero_row,
	     current_node->zero_column - 1,
	     current_node);
	current_node->zero_column--;
}

void expand(struct node *selected_node) {
	int blank_row = selected_node->zero_row;
	int blank_col = selected_node->zero_column;
	int succ_index = 0;

	for (int i = 0; i < 4; i++) {
		successor_nodes[i] = NULL;
	}

	if (blank_row > 0) {
		struct node *child = (struct node *) malloc(sizeof(struct node));
		if (child != NULL) {
			memcpy(child, selected_node, sizeof(struct node));
			child->parent = selected_node;
			child->next = NULL;
			move_up(child);
			successor_nodes[succ_index++] = child;
		}
	}

	if (blank_row < N - 1) {
		struct node *child = (struct node *) malloc(sizeof(struct node));
		if (child != NULL) {
			memcpy(child, selected_node, sizeof(struct node));
			child->parent = selected_node;
			child->next = NULL;
			move_down(child);
			successor_nodes[succ_index++] = child;
		}
	}

	if (blank_col > 0) {
		struct node *child = (struct node *) malloc(sizeof(struct node));
		if (child != NULL) {
			memcpy(child, selected_node, sizeof(struct node));
			child->parent = selected_node;
			child->next = NULL;
			move_left(child);
			successor_nodes[succ_index++] = child;
		}
	}

	if (blank_col < N - 1) {
		struct node *child = (struct node *) malloc(sizeof(struct node));
		if (child != NULL) {
			memcpy(child, selected_node, sizeof(struct node));
			child->parent = selected_node;
			child->next = NULL;
			move_right(child);
			successor_nodes[succ_index++] = child;
		}
	}
}

int nodes_same(struct node *first, struct node *second) {
	int flag = FALSE;

	if (memcmp(first->tiles, second->tiles, sizeof(int) * NxN) == 0)
		flag = TRUE;

	return flag;
}

void filter(int index, struct node *node_list) { 
	struct node *candidate = successor_nodes[index];

	if (candidate == NULL) {
		return;
	}

	while (node_list != NULL) {
		if (nodes_same(candidate, node_list)) {
			free(candidate);
			successor_nodes[index] = NULL;
			return;
		}
		node_list = node_list->next;
	}
}

int main(int argc, char **argv) {
	int iteration_count, count_unused;
	struct node *open_current, *cp_unused, *solution_path;
	int ret_unused, i, pathlen = 0, index_buffer[N - 1];

	solution_path = NULL;
	start_state = initialize(argv);	
	open_list = start_state; 

	iteration_count = 0; 
	while (open_list != NULL) {
		open_current = open_list;
		open_list = open_list->next; 

		if (nodes_same(open_current, goal_state)) { 
			do { 
				open_current->next = solution_path;
				solution_path = open_current;
				open_current = open_current->parent;
				pathlen++;
			} while (open_current != NULL);

			printf("Path (lengh=%d):\n", pathlen); 
			open_current = solution_path;

			int step = 0;
			while (open_current != NULL) {
				printf("step %d: \n", step);
				print_a_node(open_current);
				open_current = open_current->next;
				step++;
			}
			
			break;
		}

		expand(open_current);

		for (i = 0; i < 4; i++) {
			filter(i, open_list);
			filter(i, closed_list);
			update_fgh(i);
		}

		merge_to_open();

		open_current->next = closed_list;
		closed_list = open_current;		

		iteration_count++;
		if (iteration_count % 1000 == 0)
			printf("iter %d\n", iteration_count);
	}
	return 0;
} 
```
- Usage example:
```bash
./puzzle 2 3 0 4 1 6 7 8 5 9 10 12 13 14 11 15

Initial state
 2  3  0  4 
 1  6  7  8 
 5  9 10 12
13 14 11 15
Goal state
 1  2  3  4 
 5  6  7  8 
 9 10 11 12
13 14 15  0
Path (lengh=10):
step 0: 
 1  2  3  4 
 5  6  7  8 
 9 10 11 12
13 14 15  0
step 1: 
 1  2  3  4 
 5  6  7  8 
 9 10 12  0
13 14 15 11
step 2: 
 1  2  3  4 
 5  6  7  8 
 9 10 12 11
13 14 15  0
step 3: 
 1  2  3  4 
 5  6  7  8 
 9 10  0 11
13 14 15 12
step 4: 
 1  2  3  4 
 5  6  7  8 
 9  0 10 11
13 14 15 12
step 5: 
 1  2  3  4 
 5  6  7  8 
 0  9 10 11
13 14 15 12
step 6: 
 1  2  3  4 
 0  5  7  8
13  9 10 11
14 15 12  6
step 7: 
 0  1  3  4 
 5  2  7  8
13  9 10 11
14 15 12  6
step 8: 
 1  0  3  4 
 5  2  7  8
13  9 10 11
14 15 12  6
step 9: 
 1  2  3  4 
 5  0  7  8
13  9 10 11
14 15 12  6
step 10: 
 1  2  3  4 
 5  6  7  8 
 9 10 11 12
13 14 15  0
```



