
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>
#include <ctype.h>

#define MAX_PROCESSES 100
#define MAX_COMMAND_LENGTH 100
#define DEFAULT_MEMORY_SIZE 1048576  // 1MB default

// Structure to represent a memory block
typedef struct Block {
    unsigned int start_address;
    unsigned int end_address;
    char process_id[10]; // Process ID (e.g., "P0", "P1", etc.)
    bool is_allocated;
    struct Block* next;
} Block;

// Global variables
Block* memory_head = NULL;
unsigned int total_memory_size = 0;
bool memory_initialized = false;

// Function prototypes
void initialize_memory(unsigned int size);
void request_memory(char* process_id, unsigned int size, char strategy);
void release_memory(char* process_id);
void compact_memory();
void report_status();
void cleanup_memory();
Block* find_block_first_fit(unsigned int size);
Block* find_block_best_fit(unsigned int size);
Block* find_block_worst_fit(unsigned int size);
void merge_adjacent_free_blocks();
Block* find_process_block(char* process_id);
void print_help();
void print_welcome_message();
unsigned int parse_memory_size(const char* str);
bool is_valid_process_id(const char* process_id);
bool is_valid_strategy(char strategy);
void parse_command(char* command);

// Initialize memory with a single free block
void initialize_memory(unsigned int size) {
    // Clean up any previous memory allocation
    cleanup_memory();
    
    // Create a new block representing all available memory
    Block* new_block = (Block*)malloc(sizeof(Block));
    new_block->start_address = 0;
    new_block->end_address = size - 1;
    strcpy(new_block->process_id, "Unused");
    new_block->is_allocated = false;
    new_block->next = NULL;
    
    memory_head = new_block;
    total_memory_size = size;
    memory_initialized = true;
    
    printf("✅ Memory initialized with %u bytes\n", size);
}

// Request memory allocation using specified strategy
void request_memory(char* process_id, unsigned int size, char strategy) {
    if (!memory_initialized) {
        printf("❌ Error: Memory not initialized. Use default memory size? (Y/N): ");
        char response;
        scanf(" %c", &response);
        if (toupper(response) == 'Y') {
            initialize_memory(DEFAULT_MEMORY_SIZE);
        } else {
            return;
        }
        // Clear the input buffer
        while (getchar() != '\n');
    }
    
    // Check if process already exists
    if (find_process_block(process_id) != NULL) {
        printf("❌ Error: Process %s already exists. Release it first with 'RL %s'.\n", process_id, process_id);
        return;
    }
    
    // Validate size
    if (size == 0) {
        printf("❌ Error: Requested size must be greater than 0.\n");
        return;
    }
    
    if (size > total_memory_size) {
        printf("❌ Error: Requested size (%u) exceeds total memory size (%u).\n", size, total_memory_size);
        return;
    }
    
    // Find a suitable block based on the strategy
    Block* suitable_block = NULL;
    char strategy_name[20] = "";
    
    switch(toupper(strategy)) {
        case 'F': // First Fit
            suitable_block = find_block_first_fit(size);
            strcpy(strategy_name, "First Fit");
            break;
        case 'B': // Best Fit
            suitable_block = find_block_best_fit(size);
            strcpy(strategy_name, "Best Fit");
            break;
        case 'W': // Worst Fit
            suitable_block = find_block_worst_fit(size);
            strcpy(strategy_name, "Worst Fit");
            break;
        default:
            printf("❌ Invalid allocation strategy. Use F (First Fit), B (Best Fit), or W (Worst Fit).\n");
            return;
    }
    
    // Check if a suitable block was found
    if (suitable_block == NULL) {
        printf("❌ Error: Not enough contiguous memory available for allocation.\n");
        printf("   Consider running compaction with the 'C' command to consolidate free memory.\n");
        return;
    }
    
    // Calculate the block size
    unsigned int block_size = suitable_block->end_address - suitable_block->start_address + 1;
    
    // If the block is exactly the size we need, just update it
    if (block_size == size) {
        strcpy(suitable_block->process_id, process_id);
        suitable_block->is_allocated = true;
    } else {
        // Create a new block for the allocated memory
        Block* new_block = (Block*)malloc(sizeof(Block));
        new_block->start_address = suitable_block->start_address;
        new_block->end_address = suitable_block->start_address + size - 1;
        strcpy(new_block->process_id, process_id);
        new_block->is_allocated = true;
        
        // Update the remaining free block
        suitable_block->start_address = new_block->end_address + 1;
        
        // Insert the new block before the suitable block
        new_block->next = suitable_block;
        
        // Update the linked list
        if (suitable_block == memory_head) {
            memory_head = new_block;
        } else {
            Block* current = memory_head;
            while (current->next != suitable_block) {
                current = current->next;
            }
            current->next = new_block;
        }
    }
    
    printf("✅ Memory allocated for process %s using %s strategy\n", process_id, strategy_name);
    printf("   Size: %u bytes, Address range: [%u:%u]\n", 
           size, 
           find_process_block(process_id)->start_address, 
           find_process_block(process_id)->end_address);
}

// Find a block using First Fit strategy
Block* find_block_first_fit(unsigned int size) {
    Block* current = memory_head;
    
    while (current != NULL) {
        if (!current->is_allocated) {
            unsigned int block_size = current->end_address - current->start_address + 1;
            if (block_size >= size) {
                return current;
            }
        }
        current = current->next;
    }
    
    return NULL; // No suitable block found
}

// Find a block using Best Fit strategy
Block* find_block_best_fit(unsigned int size) {
    Block* current = memory_head;
    Block* best_block = NULL;
    unsigned int smallest_suitable_size = total_memory_size + 1; // Initialize to an impossible value
    
    while (current != NULL) {
        if (!current->is_allocated) {
            unsigned int block_size = current->end_address - current->start_address + 1;
            if (block_size >= size && block_size < smallest_suitable_size) {
                smallest_suitable_size = block_size;
                best_block = current;
            }
        }
        current = current->next;
    }
    
    return best_block;
}

// Find a block using Worst Fit strategy
Block* find_block_worst_fit(unsigned int size) {
    Block* current = memory_head;
    Block* worst_block = NULL;
    unsigned int largest_suitable_size = 0;
    
    while (current != NULL) {
        if (!current->is_allocated) {
            unsigned int block_size = current->end_address - current->start_address + 1;
            if (block_size >= size && block_size > largest_suitable_size) {
                largest_suitable_size = block_size;
                worst_block = current;
            }
        }
        current = current->next;
    }
    
    return worst_block;
}

// Find a block associated with a specific process
Block* find_process_block(char* process_id) {
    Block* current = memory_head;
    
    while (current != NULL) {
        if (current->is_allocated && strcmp(current->process_id, process_id) == 0) {
            return current;
        }
        current = current->next;
    }
    
    return NULL; // Process not found
}

// Release memory allocated to a process
void release_memory(char* process_id) {
    if (!memory_initialized) {
        printf("❌ Error: Memory not initialized.\n");
        return;
    }
    
    Block* block_to_release = find_process_block(process_id);
    
    if (block_to_release == NULL) {
        printf("❌ Error: Process %s not found in memory.\n", process_id);
        return;
    }
    
    unsigned int freed_size = block_to_release->end_address - block_to_release->start_address + 1;
    unsigned int start_addr = block_to_release->start_address;
    unsigned int end_addr = block_to_release->end_address;
    
    // Mark the block as free
    block_to_release->is_allocated = false;
    strcpy(block_to_release->process_id, "Unused");
    
    // Merge adjacent free blocks
    merge_adjacent_free_blocks();
    
    printf("✅ Memory released for process %s\n", process_id);
    printf("   Freed %u bytes from address range [%u:%u]\n", freed_size, start_addr, end_addr);
}

// Merge adjacent free blocks
void merge_adjacent_free_blocks() {
    bool merged;
    
    do {
        merged = false;
        Block* current = memory_head;
        
        while (current != NULL && current->next != NULL) {
            if (!current->is_allocated && !current->next->is_allocated) {
                // Merge with next block
                current->end_address = current->next->end_address;
                Block* to_delete = current->next;
                current->next = current->next->next;
                free(to_delete);
                merged = true;
            } else {
                current = current->next;
            }
        }
    } while (merged);
}

// Compact memory to consolidate all free blocks
void compact_memory() {
    if (!memory_initialized) {
        printf("❌ Error: Memory not initialized.\n");
        return;
    }
    
    if (memory_head == NULL) {
        printf("❌ Error: Memory empty or corrupted.\n");
        return;
    }
    
    // Count allocated blocks and total free memory before compaction
    unsigned int allocated_count = 0;
    unsigned int total_free_memory = 0;
    unsigned int free_blocks_count = 0;
    
    Block* current = memory_head;
    while (current != NULL) {
        if (current->is_allocated) {
            allocated_count++;
        } else {
            free_blocks_count++;
            total_free_memory += (current->end_address - current->start_address + 1);
        }
        current = current->next;
    }
    
    if (free_blocks_count <= 1) {
        printf("ℹ️ Memory is already optimally arranged. No compaction needed.\n");
        return;
    }
    
    // Create a temporary list to store allocated blocks
    Block* allocated_blocks = NULL;
    current = memory_head;
    
    // Extract all allocated blocks
    while (current != NULL) {
        Block* next = current->next;
        
        if (current->is_allocated) {
            // Remove from original list and add to allocated_blocks
            if (current == memory_head) {
                memory_head = memory_head->next;
            } else {
                Block* prev = memory_head;
                while (prev != NULL && prev->next != current) {
                    prev = prev->next;
                }
                if (prev != NULL) {
                    prev->next = current->next;
                }
            }
            
            current->next = allocated_blocks;
            allocated_blocks = current;
        }
        
        current = next;
    }
    
    // Create a single free block representing all memory
    Block* free_block = (Block*)malloc(sizeof(Block));
    free_block->start_address = 0;
    free_block->end_address = total_memory_size - 1;
    strcpy(free_block->process_id, "Unused");
    free_block->is_allocated = false;
    free_block->next = NULL;
    
    memory_head = free_block;
    
    // Reinsert allocated blocks at the beginning of memory
    unsigned int current_address = 0;
    
    while (allocated_blocks != NULL) {
        Block* block = allocated_blocks;
        allocated_blocks = allocated_blocks->next;
        
        unsigned int block_size = block->end_address - block->start_address + 1;
        
        // Update addresses
        block->start_address = current_address;
        block->end_address = current_address + block_size - 1;
        current_address += block_size;
        
        // Insert at the beginning of memory_head
        block->next = memory_head;
        memory_head = block;
    }
    
    // Update the free block
    Block* last_allocated = NULL;
    current = memory_head;
    
    while (current != NULL) {
        if (current->is_allocated) {
            last_allocated = current;
        }
        current = current->next;
    }
    
    if (last_allocated != NULL) {
        free_block->start_address = last_allocated->end_address + 1;
        
        // Remove the free block if there's no space left
        if (free_block->start_address > free_block->end_address) {
            current = memory_head;
            if (current == free_block) {
                memory_head = memory_head->next;
            } else {
                while (current->next != free_block) {
                    current = current->next;
                }
                current->next = free_block->next;
            }
            free(free_block);
        }
    }
    
    // Merge adjacent free blocks (should be only one or none after compaction)
    merge_adjacent_free_blocks();
    
    printf("✅ Memory compacted successfully.\n");
    printf("   Consolidated %d free blocks into one continuous block of %u bytes.\n", 
           free_blocks_count, total_free_memory);
}

// Report memory status
void report_status() {
    if (!memory_initialized) {
        printf("❌ Error: Memory not initialized.\n");
        return;
    }
    
    Block* current = memory_head;
    
    if (current == NULL) {
        printf("ℹ️ Memory is empty or not properly initialized.\n");
        return;
    }
    
    // Calculate statistics
    unsigned int total_used = 0;
    unsigned int total_free = 0;
    unsigned int allocated_blocks = 0;
    unsigned int free_blocks = 0;
    
    Block* temp = memory_head;
    while (temp != NULL) {
        unsigned int block_size = temp->end_address - temp->start_address + 1;
        if (temp->is_allocated) {
            total_used += block_size;
            allocated_blocks++;
        } else {
            total_free += block_size;
            free_blocks++;
        }
        temp = temp->next;
    }
    
    // Print memory map
    printf("\n=== MEMORY MAP ===\n");
    printf("Total memory: %u bytes | Used: %u bytes (%.1f%%) | Free: %u bytes (%.1f%%)\n", 
           total_memory_size, 
           total_used, 
           (float)total_used / total_memory_size * 100,
           total_free,
           (float)total_free / total_memory_size * 100);
    
    printf("\n%-12s %-15s %-15s %-10s\n", "REGION", "START", "END", "SIZE");
    printf("------------------------------------------------------\n");
    
    while (current != NULL) {
        unsigned int block_size = current->end_address - current->start_address + 1;
        printf("%-12s %-15u %-15u %-10u\n", 
               current->is_allocated ? current->process_id : "Unused", 
               current->start_address, 
               current->end_address,
               block_size);
        current = current->next;
    }
    
    printf("------------------------------------------------------\n");
    printf("Process count: %u | Free regions: %u\n\n", allocated_blocks, free_blocks);
}

// Clean up memory
void cleanup_memory() {
    Block* current = memory_head;
    
    while (current != NULL) {
        Block* to_delete = current;
        current = current->next;
        free(to_delete);
    }
    
    memory_head = NULL;
    memory_initialized = false;
}

// Print help message
void print_help() {
    printf("\n=== MEMORY ALLOCATOR HELP ===\n");
    printf("Available commands:\n");
    printf("  INIT <size>     - Initialize memory with specified size\n");
    printf("  RQ <id> <size> <strategy> - Request memory allocation\n");
    printf("                    <id>: Process ID (e.g., P0, P1)\n");
    printf("                    <size>: Memory size in bytes\n");
    printf("                    <strategy>: F (First Fit), B (Best Fit), W (Worst Fit)\n");
    printf("  RL <id>         - Release memory allocated to process\n");
    printf("  C               - Compact memory (consolidate free blocks)\n");
    printf("  STAT            - Display memory status\n");
    printf("  HELP            - Display this help message\n");
    printf("  X               - Exit program\n\n");
    printf("Examples:\n");
    printf("  INIT 1048576    - Initialize with 1MB of memory\n");
    printf("  RQ P0 50000 F   - Allocate 50000 bytes to P0 using First Fit\n");
    printf("  RL P0           - Release memory allocated to P0\n");
    printf("  STAT            - Show current memory status\n");
}

// Print welcome message
void print_welcome_message() {
    printf("===================================================\n");
    printf("           MEMORY ALLOCATOR SIMULATOR\n");
    printf("===================================================\n");
    printf("This program simulates contiguous memory allocation\n");
    printf("with different allocation strategies.\n\n");
    printf("Type 'HELP' for available commands.\n");
    printf("===================================================\n\n");
}

// Parse memory size with support for KB, MB units
unsigned int parse_memory_size(const char* str) {
    char* endptr;
    unsigned int size = strtoul(str, &endptr, 10);
    
    if (*endptr != '\0') {
        // Check for KB or MB suffix
        if (strcasecmp(endptr, "KB") == 0 || strcasecmp(endptr, "K") == 0) {
            size *= 1024;
        } else if (strcasecmp(endptr, "MB") == 0 || strcasecmp(endptr, "M") == 0) {
            size *= 1024 * 1024;
        } else {
            return 0; // Invalid suffix
        }
    }
    
    return size;
}

// Validate process ID
bool is_valid_process_id(const char* process_id) {
    // Process ID should start with 'P' followed by at least one character
    if (process_id == NULL || strlen(process_id) < 2 || toupper(process_id[0]) != 'P') {
        return false;
    }
    
    return true;
}

// Validate allocation strategy
bool is_valid_strategy(char strategy) {
    strategy = toupper(strategy);
    return (strategy == 'F' || strategy == 'B' || strategy == 'W');
}

// Parse and execute command
void parse_command(char* command) {
    char cmd[MAX_COMMAND_LENGTH];
    char arg1[MAX_COMMAND_LENGTH];
    char arg2[MAX_COMMAND_LENGTH];
    char arg3[MAX_COMMAND_LENGTH];
    
    // Default values
    cmd[0] = '\0';
    arg1[0] = '\0';
    arg2[0] = '\0';
    arg3[0] = '\0';
    
    // Parse command and arguments
    sscanf(command, "%s %s %s %s", cmd, arg1, arg2, arg3);
    
    // Convert command to uppercase for case-insensitive comparison
    for (int i = 0; cmd[i]; i++) {
        cmd[i] = toupper(cmd[i]);
    }
    
    // Process commands
    if (strcmp(cmd, "INIT") == 0) {
        if (arg1[0] == '\0') {
            printf("❌ Error: Missing memory size. Usage: INIT <size>\n");
            return;
        }
        
        unsigned int size = parse_memory_size(arg1);
        if (size == 0) {
            printf("❌ Error: Invalid memory size. Use a positive number, optionally with KB or MB suffix.\n");
            return;
        }
        
        initialize_memory(size);
    } else if (strcmp(cmd, "RQ") == 0 || strcmp(cmd, "REQUEST") == 0) {
        if (arg1[0] == '\0' || arg2[0] == '\0' || arg3[0] == '\0') {
            printf("❌ Error: Missing arguments. Usage: RQ <process_id> <size> <strategy>\n");
            return;
        }
        
        if (!is_valid_process_id(arg1)) {
            printf("❌ Error: Invalid process ID. Should start with 'P' (e.g., P0, P1).\n");
            return;
        }
        
        unsigned int size = parse_memory_size(arg2);
        if (size == 0) {
            printf("❌ Error: Invalid memory size. Use a positive number, optionally with KB or MB suffix.\n");
            return;
        }
        
        if (!is_valid_strategy(arg3[0])) {
            printf("❌ Error: Invalid strategy. Use F (First Fit), B (Best Fit), or W (Worst Fit).\n");
            return;
        }
        
        request_memory(arg1, size, arg3[0]);
    } else if (strcmp(cmd, "RL") == 0 || strcmp(cmd, "RELEASE") == 0) {
        if (arg1[0] == '\0') {
            printf("❌ Error: Missing process ID. Usage: RL <process_id>\n");
            return;
        }
        
        release_memory(arg1);
    } else if (strcmp(cmd, "C") == 0 || strcmp(cmd, "COMPACT") == 0) {
        compact_memory();
    } else if (strcmp(cmd, "STAT") == 0 || strcmp(cmd, "STATUS") == 0) {
        report_status();
    } else if (strcmp(cmd, "HELP") == 0 || strcmp(cmd, "?") == 0) {
        print_help();
    } else if (strcmp(cmd, "X") == 0 || strcmp(cmd, "EXIT") == 0 || strcmp(cmd, "QUIT") == 0) {
        printf("Exiting Memory Allocator. Goodbye!\n");
        cleanup_memory();
        exit(0);
    } else if (cmd[0] == '\0') {
        // Empty command, do nothing
    } else {
        printf("❌ Unknown command: %s. Type 'HELP' for available commands.\n", cmd);
    }
}

int main(int argc, char *argv[]) {
    // Welcome message
    print_welcome_message();
    
    // Initialize memory if size is provided as command-line argument
    if (argc == 2) {
        unsigned int memory_size = parse_memory_size(argv[1]);
        if (memory_size > 0) {
            initialize_memory(memory_size);
        } else {
            printf("❌ Invalid memory size. Using default size of 1MB.\n");
            initialize_memory(DEFAULT_MEMORY_SIZE);
        }
    }
    
    char command[MAX_COMMAND_LENGTH];
    
    while (1) {
        printf("allocator> ");
        if (fgets(command, MAX_COMMAND_LENGTH, stdin) == NULL) {
            break; // Handle EOF or error
        }
        
        // Remove trailing newline
        command[strcspn(command, "\n")] = 0;
        
        // Parse and execute command
        parse_command(command);
    }
    
    // Clean up before exiting
    cleanup_memory();
    
    return 0;
}
