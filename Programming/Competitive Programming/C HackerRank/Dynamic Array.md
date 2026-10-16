`int* dynamicArray(int n, int queries_rows, int queries_columns, int** queries, int* result_count) {`
    `int** sequences = malloc(n * sizeof(int*));       // arr[0..n-1] = int arrays`
    `int* sizes = calloc(n, sizeof(int));              // sizes of each arr[i]`
    `int* capacities = calloc(n, sizeof(int));         // capacities of each arr[i]`
    
    for (int i = 0; i < n; i++) {
        sequences[i] = NULL;
    }

    int lastAnswer = 0;
    int* results = malloc(queries_rows * sizeof(int)); // Max result size = q
    int res_size = 0;

    for (int i = 0; i < queries_rows; i++) {
        int type = queries[i][0];
        int x = queries[i][1];
        int y = queries[i][2];

        int idx = (x ^ lastAnswer) % n;

        if (type == 1) {
            // Append y to sequences[idx]
            if (sizes[idx] == capacities[idx]) {
                capacities[idx] = capacities[idx] == 0 ? 1 : capacities[idx] * 2;
                sequences[idx] = realloc(sequences[idx], capacities[idx] * sizeof(int));
            }
            sequences[idx][sizes[idx]++] = y;

        } else if (type == 2) {
            int size = sizes[idx];
            int value = sequences[idx][y % size];
            lastAnswer = value;
            results[res_size++] = lastAnswer;
        }
    }

    // Set final result count
    *result_count = res_size;

    // Free intermediate memory (not required for HackerRank, but good practice)
    for (int i = 0; i < n; i++) {
        free(sequences[i]);
    }
    free(sequences);
    free(sizes);
    free(capacities);

    return results;
}
