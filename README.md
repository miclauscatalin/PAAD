

#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
#include <time.h>

int main(int argc, char *argv[]) {
    int rank, size;
    int x0, x1, x2, x3;
    int num_received;

    // Inițializarea MPI
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Asigurăm că avem cel puțin 4 procese
    if (size < 4) {
        if (rank == 0) {
            printf("Este nevoie de cel puțin 4 procese!\n");
        }
        MPI_Finalize();
        return 1;
    }

    // Generarea unui număr aleator între 1 și 9 (cifra nenulă)
    srand(time(NULL) + rank); // Se adaugă rank-ul pentru a obține diferite secvențe de numere aleatoare
    int digit = rand() % 9 + 1; // Rand generează un număr între 0 și 8, adăugăm 1 pentru a-l face între 1 și 9
    
    // Procesul 0
    if (rank == 0) {
        x0 = digit;
        printf("Procesul 0 generează cifra: %d\n", x0);
        MPI_Send(&x0, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
        MPI_Recv(&x3, 1, MPI_INT, 3, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        
        printf("Procesul 0 afișează numărul final: %d\n", x3);
    }
    // Procesul 1
    else if (rank == 1) {
        MPI_Recv(&x0, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        x1 = digit;
        printf("Procesul 1 generează cifra: %d\n", x1);
        int num12 = x1 * 10 + x0;  // Construieste numarul x1x0
        printf("Procesul 1 construieste numarul: %d\n", num12);
        MPI_Send(&num12, 1, MPI_INT, 2, 0, MPI_COMM_WORLD);
    }
    // Procesul 2
    else if (rank == 2) {
        MPI_Recv(&x1, 1, MPI_INT, 1, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        x2 = digit;
        printf("Procesul 2 generează cifra: %d\n", x2);
        int num23 = x2 * 100 + x1 ;  // Construieste numarul x2x1x0
        printf("Procesul 2 construieste numarul: %d\n", num23);
        MPI_Send(&num23, 1, MPI_INT, 3, 0, MPI_COMM_WORLD);
    }
    // Procesul 3
    else if (rank == 3) {
        MPI_Recv(&x2, 1, MPI_INT, 2, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        x3 = digit;
        printf("Procesul 3 generează cifra: %d\n", x3);
        int num34 = x3 * 1000 + x2;  // Construieste numarul x3x2x1x0
        printf("Procesul 3 construieste numarul: %d\n", num34);
        MPI_Send(&num34, 1, MPI_INT, 0, 0, MPI_COMM_WORLD);
    }

    // Procesul 0 afișează numărul primit de la procesul 3
    if (rank == 0) {
        MPI_Recv(&x3, 1, MPI_INT, 3, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        int final_num = x3 * 1000 + x2 * 100 + x1 * 10 + x0;
        printf("Procesul 0 afișează numărul final: %d\n", final_num);
    }

    // Finalizarea MPI
    MPI_Finalize();
    return 0;
}
