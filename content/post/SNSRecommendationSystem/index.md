---
title: sns
description: Useful shortcodes that can be used in Markdown
date: 2025-08-25 00:00:00+0000
image: sns.jpg
links:
  - title: GitHub
    description: GitHub is the world's largest software development platform.
    website: https://github.com
    image: https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png
hidden: false
comments: true
draft: true
---

## SNS 친구 추천 시스템
> sns에서 사용자들 간의 친구 관계를 그래프로 표현하고, 특정 사용자에게 새로운 친구를 추천하는 프로그램

### 요구사항
 - 무방향 그래프로 친구 관계를 표현
 - 특정 사용자의 친구가 아니면서, 공통 친구가 가장 많은 사용자들을 추천
 - 추천 대상은 공통 친구 수가 많은 순서대로 정렬
 - 단, 공통 친구 수가 같다면 사용자 ID 오름차순으로 정렬

### 주의사항
 - 그래프는 인접 리스트로 표현해야 한다.
 - 추천 친구가 없을 경우에는 "No Friend"라고 출력해야 한다.
 - 프로그램의 실행은 명령줄에서 아래와 같이 실행되어야 한다.

### 코드
#### graph.h
```c
#ifndef _GRAPH_
#define _GRAPH_

#define TRUE 1
#define FALSE 0
#define MAX_VERTICES 50

typedef struct GraphNode
{
	int vertex;
	struct GraphNode* link;
} GraphNode;

typedef struct GraphType {
	int n;	
	GraphNode* adj_list[MAX_VERTICES];
} GraphType;

void graph_init(GraphType* g);
void insert_vertex(GraphType* g, int v);
void insert_edge(GraphType* g, int u, int v);
void Findfriend(GraphType* g, int id, int n);
int Checknode(GraphType* g, int id, int n);

#endif // !_GRAPH_
```

#### graph.c
```c
#define _CRT_SECURE_NO_WARNINGS 
#define _CRT_NONSTDC_NO_DEPRECATE
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#include <stdlib.h>
#include <math.h>
#include <time.h>
#include <malloc.h>

#include "graph.h"

void graph_init(GraphType* g)
{
	int v;
	g->n = 0;
	for (v = 0; v < MAX_VERTICES; v++)
		g->adj_list[v] = NULL;
}
void insert_vertex(GraphType* g, int v)
{
	if (((g->n) + 1) > MAX_VERTICES) {
		fprintf(stderr, "그래프: 정점의 개수 초과");
		return;
	}
	g->n++;
}
void insert_edge(GraphType* g, int u, int v)
{
	GraphNode* node;
	if (u >= g->n || v >= g->n) {
		fprintf(stderr, "그래프: 정점 번호 오류");
		return;
	}
	node = (GraphNode*)malloc(sizeof(GraphNode));
	node->vertex = v;
	node->link = g->adj_list[u];
	g->adj_list[u] = node;
}


// 친구를 찾아서 추천해주는 코드
void Findfriend(GraphType* g, int id, int n){
	id -= 1;
	int *A = (int*)malloc(n * sizeof(int));
	int *B = (int*)malloc(n * sizeof(int));
	for (int i = 0; i < n; i++){
		A[i] = 0;
		B[i] = 0;
	}
	// id와 연결되어 있는 노드들을 탐색
	GraphNode* node = g->adj_list[id];
	for (;node != NULL; node = node->link){
		GraphNode* _node = g->adj_list[node->vertex];
		while (_node != NULL){
			if (_node->link != NULL && _node->vertex != id) {
				// 노드가 겹치는 지 확인
				if (Checknode(g, id, _node->vertex)){
					// B 배열에 인덱스를 저장
					B[_node->vertex] = _node->vertex;
					// A 배열에 해당 인덱스의 횟수 카운트
					A[_node->vertex] += 1;
				}
			}
			_node = _node->link;
		}
	}
	// 정렬
	for (int i = 0; i < n - 1; i++){
		for (int j = i + 1; j < n; j++){
			if (B[i] != 0){
				// 공통 친구 수가 많은 순서대로 정렬
				if (A[B[i]] < A[B[i+1]]){
					int temp = B[i];
					B[i] = B[i+1];
					B[i+1] = temp;
				}
				// 사용자 ID 오름차순으로 정렬
				if (A[B[i]] == A[B[i+1]]){
					if (B[i] > B[i+1]){
						int temp = B[i];
						B[i] = B[i+1];
						B[i+1] = temp;
					}
				}
			}
		}
	}
	int sum = 0;
	for (int i = 0; i < n; i++){
		if (A[i] != 0) printf("%d %d\n", B[i] + 1, A[i]);
		sum += A[i];
	}
	// 추천 친구가 없다면 "No Friend" 출력
	if (sum == 0) printf("No friend\n");
	free(A);
	free(B);
}
// 겹치는 노드의 여부를 체크
int Checknode(GraphType* g, int id, int n) {
    GraphNode* checkNode = g->adj_list[id];
    while (checkNode != NULL) {
        if (checkNode->vertex == n) {
            return 0; 
        }
        checkNode = checkNode->link;
    }
    return 1;
}
```

#### main.c
``` c
#define _CRT_SECURE_NO_WARNINGS 
#define _CRT_NONSTDC_NO_DEPRECATE
#include <stdio.h>
#include <string.h>
#include <ctype.h>
#include <stdlib.h>
#include <math.h>
#include <time.h>
#include <malloc.h>

#include "graph.h"

int main(int argc, char *argv[])
{
	if (argc != 2) {
		fprintf(stderr, "Usage: %s graph.txt\n", argv[0]);
		exit(EXIT_FAILURE);
	}
	const char* fname = argv[1];
	FILE* fin = fopen(fname, "r");
	if (fin == NULL) {
		fprintf(stderr, "Can`t open %s\n", fname);
		exit(EXIT_FAILURE);
	}
	GraphType g;
	graph_init(&g);
	int n, m, id, fr, to;
	fscanf(fin, "%d %d %d", &n, &m, &id);
	for (int i = 0; i < n; i++) insert_vertex(&g, i);
	for (int i = 0; i < m; i++) {
		fscanf(fin, "%d %d", &fr, &to);
		insert_edge(&g, fr - 1, to - 1);
		insert_edge(&g, to - 1, fr - 1);

	}
	Findfriend(&g, id, n);
	fclose(fin);
	return 0;
}
```