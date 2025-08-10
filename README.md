#include <stdio.h>

#define MEMORY_SIZE 102400  // 100KB

// Simulated memory block
char memory[MEMORY_SIZE];

// Block header structure
typedef struct BlockHeader {
    int size;                  // Size of the block (excluding header)
    int is_free;               // 1 if free, 0 if allocated
    struct BlockHeader* next; // Next block in the list
} BlockHeader;

// Head of the free list
BlockHeader* free_list = NULL;

// Align size to 4 bytes
int align4(int size) {
    return (size + 3) & ~3;
}

// Initialize the memory allocator
void initialize_memory() {
    free_list = (BlockHeader*)memory;
    free_list->size = MEMORY_SIZE - sizeof(BlockHeader);
    free_list->is_free = 1;
    free_list->next = NULL;
}

// Allocate memory
void* allocate(int size) {
    if (size <= 0 || size > MEMORY_SIZE - sizeof(BlockHeader))
        return NULL;

    size = align4(size);
    BlockHeader* curr = free_list;

    while (curr) {
        if (curr->is_free && curr->size >= size) {
            if (curr->size >= size + sizeof(BlockHeader) + 4) {
                // Split the block
                BlockHeader* new_block = (BlockHeader*)((char*)curr + sizeof(BlockHeader) + size);
                new_block->size = curr->size - size - sizeof(BlockHeader);
                new_block->is_free = 1;
                new_block->next = curr->next;

                curr->size = size;
                curr->is_free = 0;
                curr->next = new_block;
            } else {
                // Allocate entire block
                curr->is_free = 0;
            }
            return (char*)curr + sizeof(BlockHeader);
        }
        curr = curr->next;
    }

    return NULL;  // No suitable block found
}

// Deallocate memory
void deallocate(void* ptr) {
    if (!ptr)
        return;

    BlockHeader* block = (BlockHeader*)((char*)ptr - sizeof(BlockHeader));
    block->is_free = 1;

    // Coalesce adjacent free blocks
    BlockHeader* curr = free_list;
    while (curr && curr->next) {
        if (curr->is_free && curr->next->is_free) {
            curr->size += sizeof(BlockHeader) + curr->next->size;
            curr->next = curr->next->next;
        } else {
            curr = curr->next;
        }
    }
}

// Test main function
int* mem[100];

int main() {
    initialize_memory();

    mem[0] = allocate(128);
    mem[1] = allocate(1024);
    mem[2] = allocate(4096);

    printf("mem[0] = %p\n", mem[0]);
    printf("mem[1] = %p\n", mem[1]);
    printf("mem[2] = %p\n", mem[2]);

    deallocate(mem[1]);

    mem[1] = allocate(512);
    printf("mem[1] after deallocation and reallocation = %p\n", mem[1]);

    return 0;
}
