## BFS

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX 100

void bfs(int adj[MAX][MAX], int V, int startVertex) {
    int queue[MAX], front = 0, rear = 0;
    bool visited[MAX] = { false };

    visited[startVertex] = true;
    queue[rear++] = startVertex;

    while (front < rear) {
        int currentVertex = queue[front++];
        printf("%d ", currentVertex);

        for (int i = 0; i < V; i++) {
            if (adj[currentVertex][i] == 1 && !visited[i]) {
                visited[i] = true;
                queue[rear++] = i;
            }
        }
    }
}

void addEdge(int adj[MAX][MAX], int u, int v) {
    adj[u][v] = 1;
    adj[v][u] = 1;
}

int main() {
    int V = 5;
    int adj[MAX][MAX] = {0};

    addEdge(adj, 0, 1);
    addEdge(adj, 0, 2);
    addEdge(adj, 1, 3);
    addEdge(adj, 1, 4);
    addEdge(adj, 2, 4);

    printf("BFS starting from vertex 0:\n");
    bfs(adj, V, 0);

    return 0;
}
```

## Binary Search

```c
#include <stdio.h>

int binSearch(int* arr, int len, int target) {
    int l = 0;
    int h = len - 1;

    while (l <= h) {
        int mid = l + (h - l) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) l = mid + 1;
        else if (arr[mid] > target) h = mid - 1;
    }
    return -1;
}

int binSearchRecc(int* arr, int l, int h, int len, int target) {
    int mid = l + (h - l) / 2;
    if (l > h) return -1;
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) return binSearchRecc(arr, mid + 1, h, len, target);
    else if (arr[mid] > target) return binSearchRecc(arr, l, mid - 1, len, target);
}

int main() {
    int arr[] = {1, 7, 12, 22, 35, 54};
    int len = sizeof(arr) / sizeof(arr[0]);
    int target = 71;
    int loopAns = binSearch(arr, len, target);
    int reccAns = binSearchRecc(arr, 0, len - 1, len, target);
    printf("%d, %d", loopAns, reccAns);

    return 0;
}
```

## Binary Search Tree (BST)

```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
    int data;
    struct Node* left;
    struct Node* right;
};

struct Node* createNode(int data) {
    struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
    newNode->data = data;
    newNode->left = newNode->right = NULL;
    return newNode;
}

struct Node* insert(struct Node* root, int data) {
    if (root == NULL) return createNode(data);
    if (data < root->data) root->left = insert(root->left, data);
    else if (data > root->data) root->right = insert(root->right, data);
    return root;
}

struct Node* search(struct Node* root, int key) {
    if (root == NULL || root->data == key) return root;
    if (root->data < key) return search(root->right, key);
    return search(root->left, key);
}

struct Node* minValueNode(struct Node* root) {
    while (root && root->left != NULL) {
        root = root->left;
    }
    return root;
}

struct Node* deleteNode(struct Node* root, int key) {
    if (root == NULL) return root;
    if (key < root->data) root->left = deleteNode(root->left, key);
    else if (key > root->data) root->right = deleteNode(root->right, key);
    else {
        if (root->left == NULL) {
            struct Node* temp = root->right;
            free(root);
            return temp;
        }
        else if (root->right == NULL) {
            struct Node* temp = root->left;
            free(root);
            return temp;
        }
        struct Node* temp = minValueNode(root->right);
        root->data = temp->data;
        root->right = deleteNode(root->right, temp->data);
    }
    return root;
}

void inorder(struct Node* root) {
    if (root != NULL) {
        inorder(root->left);
        printf("%d ", root->data);
        inorder(root->right);
    }
}

int main() {
    struct Node* root = NULL;
    root = insert(root, 50);
    insert(root, 30);
    insert(root, 20);
    insert(root, 40);
    insert(root, 70);
    insert(root, 60);
    insert(root, 80);

    printf("Inorder traversal of the given tree: ");
    inorder(root);
    printf("\n");

    printf("Delete 20\n");
    root = deleteNode(root, 20);
    printf("Inorder traversal of the modified tree: ");
    inorder(root);
    printf("\n");

    printf("Delete 30\n");
    root = deleteNode(root, 30);
    printf("Inorder traversal of the modified tree: ");
    inorder(root);
    printf("\n");

    printf("Delete 50\n");
    root = deleteNode(root, 50);
    printf("Inorder traversal of the modified tree: ");
    inorder(root);

    return 0;
}
```

## Bubble Sort

```c
#include <stdio.h>

void bubbleSort(int* arr, int len) {
    for (int i = 0; i < len - 1; i++) {
        for (int j = 0; j < len - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}

int main() {
    int arr[] = {1, 35, 7, 54, 22, 12};
    int len = sizeof(arr) / sizeof(arr[0]);
    bubbleSort(arr, len);

    for (int i = 0; i < len; i++) {
        printf("%d, ", arr[i]);
    }
    printf("\n");

    return 0;
}
```

## Insertion Sort

```c
#include <stdio.h>

void insertionSort(int* arr, int len) {
    for (int i = 1; i < len; i++) {
        int key = arr[i];
        int j = i - 1;

        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

int main() {
    int arr[] = {1, 35, 7, 54, 22, 12};
    int len = sizeof(arr) / sizeof(arr[0]);
    insertionSort(arr, len);

    for (int i = 0; i < len; i++) {
        printf("%d, ", arr[i]);
    }
    printf("\n");

    return 0;
}
```

## Selection Sort

```c
#include <stdio.h>

void selectionSort(int* arr, int len) {
    for (int i = 0; i < len; i++) {
        int min_idx = i;
        for (int j = i + 1; j < len; j++) {
            if (arr[j] < arr[min_idx]) min_idx = j;
        }
        int temp = arr[min_idx];
        arr[min_idx] = arr[i];
        arr[i] = temp;
    }
}

int main() {
    int arr[] = {1, 35, 7, 54, 22, 12};
    int len = sizeof(arr) / sizeof(arr[0]);
    selectionSort(arr, len);
    for (int i = 0; i < len; i++) {
        printf("%d, ", arr[i]);
    }
    printf("\n");

    return 0;
}
```
```
