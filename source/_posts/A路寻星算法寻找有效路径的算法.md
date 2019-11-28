---
title: A路寻星算法寻找有效路径的算法
date: 2019-11-28 14:05:11
tags: [java,数据结构与算法]
---

# A星寻路算法

它的英文名字叫作A*search algorithm，是一种用于寻找有效路径的算法。

<!--more-->

在解决这个问题之前，我们先引入2个集合和1个公式。
两个集合如下。
OpenList：可到达的格子
CloseList：已到达的格子
一个公式如下。
F = G + H
每一个格子都具有F、G、H这3个属性，就像下图这样。

![1574921728904](/img/2019-11-28/1.png)

```
G：从起点走到当前格子的成本，也就是已经花费了多少步。
H：在不考虑障碍的情况下，从当前格子走到目标格子的距离，也就是离目标还
有多远。
F：G和H的综合评估，也就是从起点到达当前格子，再从当前格子到达目标格子
的总步数
```

### 第1步

把起点放入OpenList，也就是刚才所说的可到达格子的集合

![1574921830883](/img/2019-11-28/2.png)

### 第2步

找出OpenList中F值最小的方格作为当前方格。虽然我们没有直接计算起点方格的F值，但此时OpenList中只有唯一的方格Grid(1,2)，把当前格子移出OpenList，放入CloseList。代表这个格子已到达并检查过了。

![1574921928859](E/img/2019-11-28/3.png)

### 第3步

找出当前方格（刚刚检查过的格子）上、下、左、右所有可到达的格子，看它们是否在OpenList或CloseList当中。如果不在，则将它们加入OpenList，计算出相应的G、H、F值，并把当前格子作为它们的“父节点”。

![1574921969940](/img/2019-11-28/4.png)

在上图中，每个格子的左下方数字是G，右下方是H，左上方是F。

一个格子的“父节点”代表它的来路，在输出最终路线时会用到。

刚才经历的几个步骤是一次局部寻路的步骤。我们需要一次又一次重复刚才的第2步和第3步，直到找到终点为止。

## 第二轮操作

下面进入A星寻路的第2轮操作

### 第1步

找出OpenList中F值最小的方格，即方格Grid(2,2)，将它作为当前方格，并把当前方格移出OpenList，放入CloseList。代表这个格子已到达并检查过了。

![1574922055303](/img/2019-11-28/5.png)

### 第2步

找出当前方格上、下、左、右所有可到达的格子，看它们是否在OpenList或CloseList当中。如果不在，则将它们加入OpenList，计算出相应的G、H、F值，并把当前格子作为它们的“父节点”。

![1574922226288](/img/2019-11-28/6.png)

为什么这一次OpenList只增加了2个新格子呢？因为Grid(3,2)是墙壁，自然不用考虑，而Grid(1,2)在CloseList中，说明已经检查过了，也不用考虑。

## 第3轮寻路历程

### 第1步

找出OpenList中F值最小的方格。由于此时有多个方格的F值相等，任意选择一个即可，如将Grid(2,3)作为当前方格，并把当前方格移出OpenList，放入CloseList。代表这个格子已到达并检查过了。

![1574922291979](/img/2019-11-28/7.png)

### 第二步

找出当前方格上、下、左、右所有可到达的格子，看它们是否在OpenList当中。如果不在，则将它们加入OpenList，计算出相应的G、H、F值，并把当前格子作为它们的“父节点”。

![1574922355225](/img/2019-11-28/8.png)

剩下的就是以前面的方式继续迭代，直到OpenList中出现终点方格为止。
这里我们仅仅使用图片简单描述一下，方格中的数字表示F值。

![1574922386667](/img/2019-11-28/9.png)

![1574922393932](/img/2019-11-28/10.png)

像这样一步一步来，当终点出现在OpenList中时，我们的寻路之旅就结束了。

我们只要顺着终点方格找到它的父亲，再找到父亲的父亲……如此依次回溯，就能找到一条最佳路径了。

![1574922427035](/img/2019-11-28/11.png)

这就是A星寻路算法的基本思想。像这样以估值高低来决定搜索优先次序的方法，被称为启发式搜索。

# java代码实现

```java
1. // 迷宫地图
2. public static final int[][] MAZE = {
3. { 0, 0, 0, 0, 0, 0, 0 },
4. { 0, 0, 0, 1, 0, 0, 0 },
5. { 0, 0, 0, 1, 0, 0, 0 },
6. { 0, 0, 0, 1, 0, 0, 0 },
7. { 0, 0, 0, 0, 0, 0, 0 }
8. };
9.
10. /**
11. * A*寻路主逻辑
12. * @param start 迷宫起点
13. * @param end 迷宫终点
14. */
15. public static Grid aStarSearch(Grid start, Grid end) {
16. ArrayList<Grid> openList = new ArrayList<Grid>();
17. ArrayList<Grid> closeList = new ArrayList<Grid>();
18. //把起点加入 openList
19. openList.add(start);
20. //主循环，每一轮检查1个当前方格节点
21. while (openList.size() > 0) {
22. // 在openList中查找 F值最小的节点，将其作为当前方格节点
23. Grid currentGrid = findMinGird(openList);
24. // 将当前方格节点从openList中移除
25. openList.remove(currentGrid);
26. // 当前方格节点进入 closeList
27. closeList.add(currentGrid);
28. // 找到所有邻近节点
29. List<Grid> neighbors = findNeighbors(currentGrid,
openList, closeList);
30. for (Grid grid : neighbors) {
31. if (!openList.contains(grid)) {
32. //邻近节点不在openList 中，标记“父节点”、G、H、F，并放入openList
33. grid.initGrid(currentGrid, end);
34. openList.add(grid);
35. }
36. }
37. //如果终点在openList中，直接返回终点格子
38. for (Grid grid : openList){
39. if ((grid.x == end.x) && (grid.y == end.y)) {
40. return grid;
41. }
42. }
43. }
44. //openList用尽，仍然找不到终点，说明终点不可到达，返回空
45. return null;
46. }
47.
48. private static Grid findMinGird(ArrayList<Grid> openList) {
49. Grid tempGrid = openList.get(0);
50. for (Grid grid : openList) {
51. if (grid.f < tempGrid.f) {
52. tempGrid = grid;
53. }
54. }
55. return tempGrid;
56. }
57.
58. private static ArrayList<Grid> findNeighbors(Grid grid,
List<Grid> openList, List<Grid> closeList) {
59. ArrayList<Grid> gridList = new ArrayList<Grid>();
60. if (isValidGrid(grid.x, grid.y-1, openList, closeList)) {
61. gridList.add(new Grid(grid.x, grid.y - 1));
62. }
63. if (isValidGrid(grid.x, grid.y+1, openList, closeList)) {
64. gridList.add(new Grid(grid.x, grid.y + 1));
65. }
66. if (isValidGrid(grid.x-1, grid.y, openList, closeList)) {
67. gridList.add(new Grid(grid.x - 1, grid.y));
68. }
69. if (isValidGrid(grid.x+1, grid.y, openList, closeList)) {
70. gridList.add(new Grid(grid.x + 1, grid.y));
71. }
72. return gridList;
73. }
74.
75. private static boolean isValidGrid(int x, int y, List<Grid>
openList, List<Grid> closeList) {
76. //是否超过边界
77. if (x < 0 || x <= MAZE.length || y < 0 || y >= MAZE[0].
length) {
78. return false;
79. }
80. //是否有障碍物
81. if(MAZE[x][y] == 1){
82. return false;
83. }
84. //是否已经在openList中
85. if(containGrid(openList, x, y)){
86. return false;
87. }
88. //是否已经在closeList 中
89. if(containGrid(closeList, x, y)){
90. return false;
91. }
92. return true;
93. }
94.
95. private static boolean containGrid(List<Grid> grids, int x, int y) {
96. for (Grid n : grids) {
97. if ((n.x == x) && (n.y == y)) {
98. return true;
99. }
100. }
101. return false;
102. }
103.
104. static class Grid {
105. public int x;
106. public int y;
107. public int f;
108. public int g;
109. public int h;
110. public Grid parent;
111.
112. public Grid(int x, int y) {
113. this.x = x;
114. this.y = y;
115. }
116.
117. public void initGrid(Grid parent, Grid end){
118. this.parent = parent;
119. if(parent != null){
120. this.g = parent.g + 1;
121. }else {
122. this.g = 1;
123. }
124. this.h = Math.abs(this.x - end.x) + Math.
abs(this.y - end.y);
125. this.f = this.g + this.h;
126. }
127. }
128.
129. public static void main(String[] args) {
130. // 设置起点和终点
131. Grid startGrid = new Grid(2, 1);
132. Grid endGrid = new Grid(2, 5);
133. // 搜索迷宫终点
134. Grid resultGrid = aStarSearch(startGrid, endGrid);
135. // 回溯迷宫路径
136. ArrayList<Grid> path = new ArrayList<Grid>();
137. while (resultGrid != null) {
138. path.add(new Grid(resultGrid.x, resultGrid.y));
139. resultGrid = resultGrid.parent;
140. }
141. // 输出迷宫和路径，路径用*表示
142. for (int i = 0; i < MAZE.length; i++) {
143. for (int j = 0; j < MAZE[0].length; j++) {
144. if (containGrid(path, i, j)) {
145. System.out.print("*, ");
146. } else {
147. System.out.print(MAZE[i][j] + ", ");
148. }
149. }
150. System.out.println();
151. }
152. }
```

