//Estacionamiento-pf
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

// Estructuras de datos
typedef struct {
    char *nombrePropietario;
    char *apellidoPropietario;
    char *placa;
    char *horaEntrada;
} Moto;

typedef struct Nodo {
    Moto moto;
    struct Nodo *siguiente;
} NodoMoto;

// Prototipos de funciones
NodoMoto *crearNodo(Moto moto);
void insertarAlFinal(NodoMoto **cabeza, Moto moto);
NodoMoto *buscarPorPlaca(NodoMoto *cabeza, const char *placa);
void ordenarPorHoraEntrada(NodoMoto *cabeza);
void liberarLista(NodoMoto *cabeza);
void imprimirLista(NodoMoto *cabeza);
void registrarNuevaMoto(NodoMoto **cabeza);

// Función principal (main)
int main() {
    NodoMoto *listaMotos = NULL;
    int opcion;

    do {
        // Menú interactivo
        printf("\nEstacionamiento de Motos para Alumnos de la FCA\n");
        printf("\n--- Menú ---\n");
        printf("1. Buscar Moto por Placa\n");
        printf("2. Ordenar Motos por Hora de Entrada\n");
        printf("3. Imprimir Lista de Motos\n");
        printf("4. Registrar Nueva Moto\n");
        printf("5. Salir\n");

        printf("Seleccione una opción: ");
        scanf("%d", &opcion);

        switch (opcion) {
            case 1: {
                char placaBuscada[20];
                printf("Ingrese la placa a buscar: ");
                scanf("%s", placaBuscada);

                NodoMoto *motoEncontrada = buscarPorPlaca(listaMotos, placaBuscada);
                if (motoEncontrada != NULL) {
                    printf("Moto encontrada:\n");
                    printf("Nombre del propietario: %s\n", motoEncontrada->moto.nombrePropietario);
                    printf("Apellido del propietario: %s\n", motoEncontrada->moto.apellidoPropietario);
                    printf("Placa: %s\n", motoEncontrada->moto.placa);
                    printf("Hora de entrada: %s\n", motoEncontrada->moto.horaEntrada);
                } else {
                    printf("Moto no encontrada.\n");
                }
                break;
            }
            case 2:
                ordenarPorHoraEntrada(listaMotos);
                printf("Motos ordenadas por hora de entrada.\n");
                break;
            case 3:
                imprimirLista(listaMotos);
                break;
            case 4:
                registrarNuevaMoto(&listaMotos);
                printf("Nueva moto registrada correctamente.\n");
                break;
            case 5:
                printf("Saliendo del programa.\n");
                break;
            default:
                printf("Opción no válida. Por favor, seleccione nuevamente.\n");
        }

    } while (opcion != 5);

    // Liberar memoria asignada antes de salir
    liberarLista(listaMotos);

    return 0;
}

// Implementación de las funciones
NodoMoto *crearNodo(Moto moto) {
    NodoMoto *nuevoNodo = (NodoMoto *)malloc(sizeof(NodoMoto));
    nuevoNodo->moto = moto;
    nuevoNodo->siguiente = NULL;
    return nuevoNodo;
}

void insertarAlFinal(NodoMoto **cabeza, Moto moto) {
    NodoMoto *nuevoNodo = crearNodo(moto);

    if (*cabeza == NULL) {
        *cabeza = nuevoNodo;
    } else {
        NodoMoto *temp = *cabeza;
        while (temp->siguiente != NULL) {
            temp = temp->siguiente;
        }
        temp->siguiente = nuevoNodo;
    }
}

NodoMoto *buscarPorPlaca(NodoMoto *cabeza, const char *placa) {
    NodoMoto *temp = cabeza;
    while (temp != NULL) {
        if (strcmp(temp->moto.placa, placa) == 0) {
            return temp;
        }
        temp = temp->siguiente;
    }
    return NULL;
}

void ordenarPorHoraEntrada(NodoMoto *cabeza) {
    NodoMoto *actual = cabeza, *min;

    while (actual != NULL) {
        min = actual;

        NodoMoto *siguiente = actual->siguiente;
        while (siguiente != NULL) {
            if (strcmp(siguiente->moto.horaEntrada, min->moto.horaEntrada) < 0) {
                min = siguiente;
            }
            siguiente = siguiente->siguiente;
        }

        if (min != actual) {
            Moto temp = actual->moto;
            actual->moto = min->moto;
            min->moto = temp;
        }

        actual = actual->siguiente;
    }
}

void liberarLista(NodoMoto *cabeza) {
    NodoMoto *actual = cabeza;
    while (actual != NULL) {
        NodoMoto *siguiente = actual->siguiente;
        free(actual->moto.nombrePropietario);
        free(actual->moto.apellidoPropietario);
        free(actual->moto.placa);
        free(actual->moto.horaEntrada);
        free(actual);
        actual = siguiente;
    }
}

void imprimirLista(NodoMoto *cabeza) {
    NodoMoto *temp = cabeza;
    while (temp != NULL) {
        printf("Nombre del propietario: %s\n", temp->moto.nombrePropietario);
        printf("Apellido del propietario: %s\n", temp->moto.apellidoPropietario);
        printf("Placa: %s\n", temp->moto.placa);
        printf("Hora de entrada: %s\n", temp->moto.horaEntrada);
        printf("\n");
        temp = temp->siguiente;
    }
}

void registrarNuevaMoto(NodoMoto **cabeza) {
    char nombre[50], apellido[50], placa[20], horaActual[9];

    // Solicitar datos al usuario
    printf("Ingrese el nombre del propietario de la moto: ");
    scanf("%s", nombre);

    printf("Ingrese el apellido del propietario de la moto: ");
    scanf("%s", apellido);

    printf("Ingrese la placa de la moto: ");
    scanf("%s", placa);

    // Obtener la hora actual del sistema
    time_t rawtime;
    struct tm *info;
    time(&rawtime);
    info = localtime(&rawtime);
    strftime(horaActual, 9, "%H:%M:%S", info);

    // Crear nueva moto y agregarla a la lista
    Moto nuevaMoto = {strdup(nombre), strdup(apellido), strdup(placa), strdup(horaActual)};
    insertarAlFinal(cabeza, nuevaMoto);

    // Guardar la nueva moto en el archivo CSV
    FILE *archivoCsv = fopen("archivo.csv", "a");
    if (archivoCsv != NULL) {
        fprintf(archivoCsv, "%s,%s,%s,%s\n", nombre, apellido, placa, horaActual);
        fclose(archivoCsv);
    } else {
        printf("Error al abrir el archivo para guardar la nueva moto.\n");
    }
}
