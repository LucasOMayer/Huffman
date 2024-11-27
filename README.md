#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Estrutura de um nï¿½ da ï¿½rvore de Huffman
struct MinHeapNode {
    char data;
    unsigned freq;
    struct MinHeapNode *left, *right;
};

// Estrutura de uma Min Heap
struct MinHeap {
    unsigned size;
    unsigned capacity;
    struct MinHeapNode** array;
};

// Funï¿½ï¿½o para criar um novo nï¿½ da ï¿½rvore de Huffman
struct MinHeapNode* newNode(char data, unsigned freq) {
    struct MinHeapNode* temp = (struct MinHeapNode*)malloc(sizeof(struct MinHeapNode));
    temp->left = temp->right = NULL;
    temp->data = data;
    temp->freq = freq;
    return temp;
}

// Funï¿½ï¿½o para criar uma Min Heap de capacidade dada
struct MinHeap* createMinHeap(unsigned capacity) {
    struct MinHeap* minHeap = (struct MinHeap*)malloc(sizeof(struct MinHeap));
    minHeap->size = 0;
    minHeap->capacity = capacity;
    minHeap->array = (struct MinHeapNode**)malloc(minHeap->capacity * sizeof(struct MinHeapNode*));
    return minHeap;
}

// Funï¿½ï¿½o para trocar dois nï¿½s da Min Heap
void swapMinHeapNode(struct MinHeapNode** a, struct MinHeapNode** b) {
    struct MinHeapNode* t = *a;
    *a = *b;
    *b = t;
}

// Funï¿½ï¿½o padrï¿½o para minHeapify
void minHeapify(struct MinHeap* minHeap, int idx) {
    int smallest = idx;
    int left = 2 * idx + 1;
    int right = 2 * idx + 2;

    if (left < minHeap->size && minHeap->array[left]->freq < minHeap->array[smallest]->freq)
        smallest = left;

    if (right < minHeap->size && minHeap->array[right]->freq < minHeap->array[smallest]->freq)
        smallest = right;

    if (smallest != idx) {
        swapMinHeapNode(&minHeap->array[smallest], &minHeap->array[idx]);
        minHeapify(minHeap, smallest);
    }
}

// Funï¿½ï¿½o para verificar se o tamanho da Min Heap ï¿½ 1
int isSizeOne(struct MinHeap* minHeap) {
    return (minHeap->size == 1);
}

// Funï¿½ï¿½o padrï¿½o para extrair o nï¿½ de valor mï¿½nimo da heap
struct MinHeapNode* extractMin(struct MinHeap* minHeap) {
    struct MinHeapNode* temp = minHeap->array[0];
    minHeap->array[0] = minHeap->array[minHeap->size - 1];
    --minHeap->size;
    minHeapify(minHeap, 0);
    return temp;
}

// Funï¿½ï¿½o para inserir um novo nï¿½ na Min Heap
void insertMinHeap(struct MinHeap* minHeap, struct MinHeapNode* minHeapNode) {
    ++minHeap->size;
    int i = minHeap->size - 1;

    while (i && minHeapNode->freq < minHeap->array[(i - 1) / 2]->freq) {
        minHeap->array[i] = minHeap->array[(i - 1) / 2];
        i = (i - 1) / 2;
    }
    minHeap->array[i] = minHeapNode;
}

// Funï¿½ï¿½o padrï¿½o para construir a Min Heap
void buildMinHeap(struct MinHeap* minHeap) {
    int n = minHeap->size - 1;
    int i;

    for (i = (n - 1) / 2; i >= 0; --i)
        minHeapify(minHeap, i);
}

// Funï¿½ï¿½o para verificar se o nï¿½ ï¿½ uma folha
int isLeaf(struct MinHeapNode* root) {
    return !(root->left) && !(root->right);
}

// Funï¿½ï¿½o para criar e construir uma Min Heap
struct MinHeap* createAndBuildMinHeap(char data[], int freq[], int size) {
    struct MinHeap* minHeap = createMinHeap(size);
    int i;

    for (i = 0; i < size; ++i)
        minHeap->array[i] = newNode(data[i], freq[i]);

    minHeap->size = size;
    buildMinHeap(minHeap);

    return minHeap;
}

// Funï¿½ï¿½o principal para construir a ï¿½rvore de Huffman
struct MinHeapNode* buildHuffmanTree(char data[], int freq[], int size) {
    struct MinHeapNode *left, *right, *top;
    struct MinHeap* minHeap = createAndBuildMinHeap(data, freq, size);

    while (!isSizeOne(minHeap)) {
        left = extractMin(minHeap);
        right = extractMin(minHeap);

        top = newNode('$', left->freq + right->freq);
        top->left = left;
        top->right = right;

        insertMinHeap(minHeap, top);
    }

    return extractMin(minHeap);
}

// Funï¿½ï¿½o para imprimir um array de tamanho n
void printArr(int arr[], int n) {
    int i;
    for (i = 0; i < n; ++i)
        printf("%d", arr[i]);
    printf("\n");
}

// Funï¿½ï¿½o para imprimir os cï¿½digos de Huffman e calcular o tamanho compactado
void printCodes(struct MinHeapNode* root, int arr[], int top, int* totalBits) {
    if (root->left) {
        arr[top] = 0;
        printCodes(root->left, arr, top + 1, totalBits);
    }

    if (root->right) {
        arr[top] = 1;
        printCodes(root->right, arr, top + 1, totalBits);
    }

    if (isLeaf(root)) {
        printf("%c: ", root->data);
        printArr(arr, top);
        *totalBits += root->freq * top; // Calcula o total de bits compactados
    }
}

// Funï¿½ï¿½o para construir a ï¿½rvore de Huffman e imprimir os cï¿½digos
void HuffmanCodes(char data[], int freq[], int size, int* totalBits) {
    struct MinHeapNode* root = buildHuffmanTree(data, freq, size);
    int arr[100], top = 0;
    *totalBits = 0;
    printCodes(root, arr, top, totalBits);
}

// Funï¿½ï¿½o para calcular a frequï¿½ncia dos caracteres
void calculateFrequency(const char* str, char* data, int* freq, int* size) {
    int count[256] = {0};
    int length = strlen(str);
    int i;

    for (i = 0; i < length; ++i) {
        count[(unsigned char)str[i]]++;
    }

    *size = 0;
    for (i = 0; i < 256; ++i) {
        if (count[i] > 0) {
            data[*size] = (char)i;
            freq[*size] = count[i];
            (*size)++;
        }
    }
}

// Funï¿½ï¿½o principal
int main() {
    const char* str = "Estrutura de dados";
    char data[256];
    int freq[256];
    int size;
    int originalBits;
    int compressedBits;

    calculateFrequency(str, data, freq, &size);

    // Calcula o tamanho original (em bits)
    originalBits = strlen(str) * 8;

    HuffmanCodes(data, freq, size, &compressedBits);

    printf("Tamanho original: %d bits\n", originalBits);
    printf("Tamanho compactado: %d bits\n", compressedBits);

    return 0;
}
