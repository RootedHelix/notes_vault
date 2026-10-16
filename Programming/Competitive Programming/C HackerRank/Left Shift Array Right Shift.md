

The modulo operation is what allows the new_index to wrap around to the beginning of the index (addition) or to the end (left)
### Left Shift

int* rotateLeft(int d, int arr_count, int* arr, int* result_count) {
*result_count = arr_count;
int* rotated = (int*)malloc(arr_count * sizeof(int));
int new_index;
for(int i = 0; i < arr_count; i++)
{
    ==new_index = (i - d + arr_count) % arr_count;==
    rotated[new_index] = arr[i];
}
return rotated;
}

### Right Shift 

new_index = (i + d) % size; 