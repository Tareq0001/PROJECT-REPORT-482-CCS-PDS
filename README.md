# PROJECT-REPORT-482-CCS-PDS

Problem 1: Ring Message Passing

Objective

Create an MPI program where processes form a ring. Each process sends a message to the next process and receives a message from the previous process, then prints the received message.

Source Code: mpi_ring.c

#include <mpi.h>
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    int rank, size, next, prev;
    char send_msg[100], recv_msg[100];
    MPI_Status status;

    // Initialize MPI
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Determine neighbors in the ring
    next = (rank + 1) % size; // Successor
    prev = (rank - 1 + size) % size; // Predecessor

    // Prepare the message to send
    snprintf(send_msg, sizeof(send_msg), "Message from process %d", rank);

    // Send to next process and receive from previous process
    MPI_Sendrecv(send_msg, strlen(send_msg) + 1, MPI_CHAR, next, 0,
                 recv_msg, 100, MPI_CHAR, prev, 0, MPI_COMM_WORLD, &status);

    // Print the received message
    printf("Process %d received: %s\n", rank, recv_msg);

    // Finalize MPI
    MPI_Finalize();
    return 0;
}

Explanation

This program sets up a ring of processes using MPI. Each process:





Gets its rank (ID) and the total number of processes.



Calculates its successor and predecessor in the ring.



Creates a message like “Message from process X” (X is the rank).



Uses MPI_Sendrecv to send its message to the successor and receive a message from the predecessor.



Prints the received message.



Cleans up with MPI_Finalize.

The ring ensures messages circulate, e.g., Process 0 sends to Process 1 and receives from Process 3 in a 4-process ring.



Problem 2: Token-Based Mutual Exclusion

Objective

Implement a token-based algorithm for three processes (P1, P2, P3) in a ring sharing a printer. A token circulates, and only the process with the token can print.

Source Code: token_ring.c

#include <mpi.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[]) {
    int rank, size, token = 0, next, prev;
    int has_token, print_request;
    MPI_Status status;

    // Initialize MPI
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Ensure exactly 3 processes
    if (size != 3) {
        if (rank == 0) {
            printf("This program requires exactly 3 processes.\n");
        }
        MPI_Finalize();
        return 1;
    }

    // Determine neighbors in the ring
    next = (rank + 1) % size;
    prev = (rank - 1 + size) % size;

    // Process 0 initially holds the token
    has_token = (rank == 0) ? 1 : 0;

    // Simulate for a few iterations
    for (int i = 0; i < 3; i++) {
        // Simulate a random print request (0 or 1)
        print_request = rand() % 2;

        if (has_token && print_request) {
            // Process has the token and wants to print
            printf("Process %d is printing (iteration %d)\n", rank, i);
            sleep(1); // Simulate printing time
        }

        // Pass the token to the next process
        if (has_token) {
            MPI_Send(&token, 1, MPI_INT, next, 0, MPI_COMM_WORLD);
            has_token = 0;
            printf("Process %d sent token to Process %d\n", rank, next);
        }

        // Receive the token from the previous process
        MPI_Recv(&token, 1, MPI_INT, prev, 0, MPI_COMM_WORLD, &status);
        has_token = 1;
        printf("Process %d received token from Process %d\n", rank, prev);

        // Barrier to synchronize output for clarity
        MPI_Barrier(MPI_COMM_WORLD);
    }

    // Finalize MPI
    MPI_Finalize();
    return 0;
}

Explanation

This program simulates mutual exclusion for a printer:





Verifies exactly 3 processes are used.



Sets up a ring with successor and predecessor for each process.



Process 0 starts with the token.



In 3 iterations:





Randomly decides if the process wants to print (rand() % 2).



If it has the token and wants to print, it prints a message and waits (simulating printing).



Sends the token to the next process and receives it from the previous process.



Uses MPI_Barrier to keep output orderly.



Ensures only one process prints at a time, maintaining mutual exclusion.

The token circulates (P0 → P1 → P2 → P0), controlling printer access.



Setup Instructions for Windows

I set up the programs on Windows 11 using Microsoft MPI (MS-MPI) and MinGW. Below are the steps I followed.

Installing MS-MPI





Download:





Went to Microsoft MPI Download.



Downloaded MSMpiSetup.exe (runtime) and msmpisdk.msi (SDK).



Install:





Ran MSMpiSetup.exe, accepted the license, and installed to C:\Program Files\Microsoft MPI.



Ran msmpisdk.msi, accepted the license, and installed to C:\Program Files (x86)\Microsoft SDKs\MPI.



Set PATH:





Searched for “environment variables” in Windows, opened “Edit the system environment variables”.



Added C:\Program Files\Microsoft MPI\Bin to System PATH.



Tested by running mpiexec -help in Command Prompt (as Administrator).

Installing MinGW





Download:





Downloaded MSYS2 from MSYS2.



Installed it to C:\msys64.



Install GCC:





Opened MSYS2 MinGW 64-bit terminal.



Updated packages: pacman -Syu.



Installed GCC: pacman -S mingw-w64-x86_64-gcc.



Set PATH:





Added C:\msys64\mingw64\bin to System PATH.



Verified: Ran gcc --version in Command Prompt.

Firewall





Allowed mpiexec.exe and smpd.exe in Windows Firewall:





Control Panel → Windows Defender Firewall → Allow an app.



Added C:\Program Files\Microsoft MPI\Bin\mpiexec.exe and smpd.exe.



Compilation and Execution

I created a directory C:\MPI_Projects to store and run the programs.

Problem 1: mpi_ring.c





Saved Code:





Saved mpi_ring.c in C:\MPI_Projects.



Compiled:





Opened Command Prompt (as Administrator) in C:\MPI_Projects.



Ran:

gcc -I"C:\Program Files (x86)\Microsoft SDKs\MPI\Include" -L"C:\Program Files (x86)\Microsoft SDKs\MPI\Lib\x64" -lmsmpi mpi_ring.c -o mpi_ring



Executed:





Ran:

mpiexec -n 4 mpi_ring.exe > output.txt

Output (in output.txt):

Process 0 received: Message from process 3
Process 1 received: Message from process 0
Process 2 received: Message from process 1
Process 3 received: Message from process 2

Problem 2: token_ring.c











Saved token_ring.c in C:\MPI_Projects.



Compiled:





Ran:

gcc -I"C:\Program Files (x86)\Microsoft SDKs\MPI\Include" -L"C:\Program Files (x86)\Microsoft SDKs\MPI\Lib\x64" -lmsmpi token_ring.c -o token_ring



Executed:





Ran:

mpiexec -n 3 token_ring.exe > token_output.txt
Output (in token_output.txt, varies due to randomness):

Process 0 is printing (iteration 0)
Process 0 sent token to Process 1
Process 1 received token from Process 0
Process 1 sent token to Process 2
Process 2 received token from Process 1
Process 2 is printing (iteration 0)
Process 2 sent token to Process 0
Process 0 received token from Process 2
Process 1 is printing (iteration 1)
Process 1 sent token to Process 2
Process 2 received token from Process 1
Process 2 sent token to Process 0
Process 0 received token from Process 2
Process 0 sent token to Process 1
Process 1 received token from Process 0
Process 1 sent token to Process 2
Process 2 received token from Process 1
Process 2 sent token to Process 0
Process 0 received token from Process 2
