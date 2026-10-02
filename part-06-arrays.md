# Part 06: Arrays
## หลักสูตร Java & Android Development - ระดับเริ่มต้น

---

## 6.1 Array คืออะไร?

Array คือโครงสร้างข้อมูลที่เก็บข้อมูลหลายค่าชนิดเดียวกันในตัวแปรเดียว มีขนาดคงที่ (fixed size) และเข้าถึงด้วย index

```
int[] scores = {85, 92, 78, 96, 88};

Index:   [0]   [1]   [2]   [3]   [4]
Value:    85    92    78    96    88
```

---

## 6.2 สร้างและใช้งาน Array

```java
public class ArrayBasics {
    public static void main(String[] args) {
        
        // วิธีที่ 1: ประกาศและกำหนดขนาด
        int[] numbers = new int[5];  // ทุก element เป็น 0 โดย default
        
        // กำหนดค่า
        numbers[0] = 10;
        numbers[1] = 20;
        numbers[2] = 30;
        numbers[3] = 40;
        numbers[4] = 50;
        
        // วิธีที่ 2: ประกาศพร้อมกำหนดค่า
        int[] scores = {85, 92, 78, 96, 88};
        
        // วิธีที่ 3: new กับ initializer
        String[] fruits = new String[]{"Apple", "Banana", "Cherry"};
        
        // ขนาดของ array
        System.out.println("ขนาด scores: " + scores.length);  // 5
        
        // เข้าถึง element
        System.out.println("scores[0] = " + scores[0]);  // 85
        System.out.println("scores[4] = " + scores[4]);  // 88
        
        // เปลี่ยนค่า
        scores[2] = 100;
        System.out.println("scores[2] after = " + scores[2]);  // 100
        
        // วน loop แสดงทั้งหมด
        System.out.println("\n--- All scores ---");
        for (int i = 0; i < scores.length; i++) {
            System.out.printf("scores[%d] = %d%n", i, scores[i]);
        }
        
        // for-each
        System.out.println("\n--- for-each ---");
        for (int score : scores) {
            System.out.println(score);
        }
        
        // Default values
        int[] ints = new int[3];      // [0, 0, 0]
        double[] dbls = new double[3]; // [0.0, 0.0, 0.0]
        boolean[] bools = new boolean[3]; // [false, false, false]
        String[] strs = new String[3]; // [null, null, null]
        
        System.out.println("\nDefault int: " + ints[0]);
        System.out.println("Default String: " + strs[0]);
        
        // ArrayIndexOutOfBoundsException
        try {
            System.out.println(scores[10]);  // Error!
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Error: index out of bounds");
        }
    }
}
```

---

## 6.3 Array Operations

```java
import java.util.Arrays;

public class ArrayOperations {
    public static void main(String[] args) {
        int[] arr = {64, 34, 25, 12, 22, 11, 90};
        
        // หาค่าสูงสุด/ต่ำสุด
        int max = arr[0], min = arr[0];
        for (int n : arr) {
            if (n > max) max = n;
            if (n < min) min = n;
        }
        System.out.println("Max: " + max);  // 90
        System.out.println("Min: " + min);  // 11
        
        // ผลรวมและค่าเฉลี่ย
        double sum = 0;
        for (int n : arr) sum += n;
        System.out.printf("Sum: %.0f, Avg: %.2f%n", sum, sum/arr.length);
        
        // Arrays.sort() - เรียงลำดับ
        int[] sorted = arr.clone();
        Arrays.sort(sorted);
        System.out.println("Sorted: " + Arrays.toString(sorted));
        
        // Arrays.toString() - แสดงผล
        System.out.println("Original: " + Arrays.toString(arr));
        
        // Arrays.fill() - เติมค่า
        int[] filled = new int[5];
        Arrays.fill(filled, 99);
        System.out.println("Filled: " + Arrays.toString(filled));
        
        // Arrays.copyOf() - copy บางส่วน
        int[] copy = Arrays.copyOf(arr, 3);  // copy 3 elements แรก
        System.out.println("Copy(3): " + Arrays.toString(copy));
        
        // Arrays.copyOfRange()
        int[] range = Arrays.copyOfRange(arr, 2, 5);  // index 2-4
        System.out.println("Range(2-4): " + Arrays.toString(range));
        
        // Arrays.equals() - เปรียบเทียบ
        int[] a = {1, 2, 3};
        int[] b = {1, 2, 3};
        int[] c = {1, 2, 4};
        System.out.println("a equals b: " + Arrays.equals(a, b));  // true
        System.out.println("a equals c: " + Arrays.equals(a, c));  // false
        
        // Binary Search (array ต้อง sort ก่อน)
        int target = 22;
        int index = Arrays.binarySearch(sorted, target);
        System.out.println("Index of " + target + ": " + index);
        
        // Reverse array (ไม่มี built-in ต้องทำเอง)
        int[] reversed = arr.clone();
        for (int i = 0; i < reversed.length / 2; i++) {
            int temp = reversed[i];
            reversed[i] = reversed[reversed.length - 1 - i];
            reversed[reversed.length - 1 - i] = temp;
        }
        System.out.println("Reversed: " + Arrays.toString(reversed));
    }
}
```

---

## 6.4 2D Arrays (อาร์เรย์ 2 มิติ)

```java
import java.util.Arrays;

public class TwoDimensionalArray {
    public static void main(String[] args) {
        
        // สร้าง 2D array
        int[][] matrix = new int[3][4];  // 3 rows, 4 cols
        
        // กำหนดค่า
        int value = 1;
        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 4; j++) {
                matrix[i][j] = value++;
            }
        }
        
        // แสดงผล
        System.out.println("=== Matrix ===");
        for (int[] row : matrix) {
            for (int val : row) {
                System.out.printf("%4d", val);
            }
            System.out.println();
        }
        
        // Shorthand initialization
        int[][] grid = {
            {1, 2, 3},
            {4, 5, 6},
            {7, 8, 9}
        };
        
        System.out.println("\n=== Grid ===");
        for (int[] row : grid) {
            System.out.println(Arrays.toString(row));
        }
        
        // ขนาด
        System.out.println("\nRows: " + grid.length);         // 3
        System.out.println("Cols: " + grid[0].length);       // 3
        
        // Matrix operations
        System.out.println("\n=== Matrix Transpose ===");
        int rows = grid.length;
        int cols = grid[0].length;
        int[][] transposed = new int[cols][rows];
        
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                transposed[j][i] = grid[i][j];
            }
        }
        
        for (int[] row : transposed) {
            System.out.println(Arrays.toString(row));
        }
        
        // Matrix addition
        System.out.println("\n=== Matrix Addition ===");
        int[][] m1 = {{1, 2}, {3, 4}};
        int[][] m2 = {{5, 6}, {7, 8}};
        int[][] sum = new int[2][2];
        
        for (int i = 0; i < 2; i++) {
            for (int j = 0; j < 2; j++) {
                sum[i][j] = m1[i][j] + m2[i][j];
            }
        }
        
        System.out.println("m1: " + Arrays.deepToString(m1));
        System.out.println("m2: " + Arrays.deepToString(m2));
        System.out.println("sum: " + Arrays.deepToString(sum));
        
        // Jagged array (แต่ละ row มีขนาดต่างกัน)
        System.out.println("\n=== Jagged Array ===");
        int[][] jagged = new int[5][];
        for (int i = 0; i < jagged.length; i++) {
            jagged[i] = new int[i + 1];
            for (int j = 0; j <= i; j++) {
                jagged[i][j] = i + j;
            }
        }
        
        for (int[] row : jagged) {
            System.out.println(Arrays.toString(row));
        }
    }
}
```

---

## 6.5 Sorting Algorithms

```java
import java.util.Arrays;

public class SortingAlgorithms {
    
    // Bubble Sort
    static void bubbleSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j+1]) {
                    int temp = arr[j];
                    arr[j] = arr[j+1];
                    arr[j+1] = temp;
                }
            }
        }
    }
    
    // Selection Sort
    static void selectionSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            int minIdx = i;
            for (int j = i + 1; j < n; j++) {
                if (arr[j] < arr[minIdx]) {
                    minIdx = j;
                }
            }
            int temp = arr[minIdx];
            arr[minIdx] = arr[i];
            arr[i] = temp;
        }
    }
    
    // Insertion Sort
    static void insertionSort(int[] arr) {
        int n = arr.length;
        for (int i = 1; i < n; i++) {
            int key = arr[i];
            int j = i - 1;
            while (j >= 0 && arr[j] > key) {
                arr[j+1] = arr[j];
                j--;
            }
            arr[j+1] = key;
        }
    }
    
    public static void main(String[] args) {
        int[] original = {64, 34, 25, 12, 22, 11, 90};
        
        System.out.println("Original: " + Arrays.toString(original));
        
        // Bubble Sort
        int[] arr1 = original.clone();
        bubbleSort(arr1);
        System.out.println("Bubble Sort: " + Arrays.toString(arr1));
        
        // Selection Sort
        int[] arr2 = original.clone();
        selectionSort(arr2);
        System.out.println("Selection Sort: " + Arrays.toString(arr2));
        
        // Insertion Sort
        int[] arr3 = original.clone();
        insertionSort(arr3);
        System.out.println("Insertion Sort: " + Arrays.toString(arr3));
        
        // Java built-in sort
        int[] arr4 = original.clone();
        Arrays.sort(arr4);
        System.out.println("Arrays.sort: " + Arrays.toString(arr4));
        
        // Sort descending
        Integer[] arr5 = {64, 34, 25, 12, 22, 11, 90};
        Arrays.sort(arr5, (a, b) -> b - a);
        System.out.println("Desc: " + Arrays.toString(arr5));
    }
}
```

---

## 6.6 Searching Algorithms

```java
import java.util.Arrays;

public class SearchingAlgorithms {
    
    // Linear Search - O(n)
    static int linearSearch(int[] arr, int target) {
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == target) return i;
        }
        return -1;  // ไม่พบ
    }
    
    // Binary Search - O(log n) (array ต้อง sort)
    static int binarySearch(int[] arr, int target) {
        int left = 0, right = arr.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (arr[mid] == target) return mid;
            else if (arr[mid] < target) left = mid + 1;
            else right = mid - 1;
        }
        return -1;
    }
    
    public static void main(String[] args) {
        int[] arr = {11, 12, 22, 25, 34, 64, 90};  // sorted
        int target = 25;
        
        System.out.println("Array: " + Arrays.toString(arr));
        System.out.println("Target: " + target);
        
        int linearIdx = linearSearch(arr, target);
        System.out.println("Linear Search: index " + linearIdx);
        
        int binaryIdx = binarySearch(arr, target);
        System.out.println("Binary Search: index " + binaryIdx);
        
        // Test not found
        int notFound = linearSearch(arr, 99);
        System.out.println("Linear Search (99): " + notFound);  // -1
        
        // Performance comparison (ทางทฤษฎี)
        System.out.println("\n--- Search Complexity ---");
        System.out.println("Linear Search: O(n) - ตรวจทีละตัว");
        System.out.println("Binary Search: O(log n) - ค้นแบบแบ่งครึ่ง");
        System.out.println("สำหรับ n=1,000,000:");
        System.out.println("  Linear: ตรวจ ~500,000 ครั้งโดยเฉลี่ย");
        System.out.println("  Binary: ตรวจ ~20 ครั้ง (log2(1000000) ≈ 20)");
    }
}
```

---

## 6.7 โปรแกรมตัวอย่าง: Student Management

```java
import java.util.Arrays;
import java.util.Scanner;

public class StudentManagement {
    static String[] names = new String[100];
    static double[] scores = new double[100];
    static int count = 0;
    
    static void addStudent(String name, double score) {
        if (count < names.length) {
            names[count] = name;
            scores[count] = score;
            count++;
            System.out.println("เพิ่ม " + name + " แล้ว");
        } else {
            System.out.println("ห้องเรียนเต็ม!");
        }
    }
    
    static void showAll() {
        if (count == 0) {
            System.out.println("ไม่มีนักเรียน");
            return;
        }
        System.out.println("\n" + "=".repeat(35));
        System.out.printf("%-5s %-15s %8s %6s%n", "No.", "ชื่อ", "คะแนน", "เกรด");
        System.out.println("-".repeat(35));
        
        for (int i = 0; i < count; i++) {
            String grade = getGrade(scores[i]);
            System.out.printf("%-5d %-15s %8.1f %6s%n", 
                (i+1), names[i], scores[i], grade);
        }
        System.out.println("=".repeat(35));
    }
    
    static String getGrade(double score) {
        if (score >= 80) return "A";
        if (score >= 70) return "B";
        if (score >= 60) return "C";
        if (score >= 50) return "D";
        return "F";
    }
    
    static void showStats() {
        if (count == 0) return;
        
        double sum = 0, max = scores[0], min = scores[0];
        for (int i = 0; i < count; i++) {
            sum += scores[i];
            if (scores[i] > max) max = scores[i];
            if (scores[i] < min) min = scores[i];
        }
        
        System.out.println("\n=== สถิติ ===");
        System.out.printf("จำนวนนักเรียน : %d คน%n", count);
        System.out.printf("ค่าเฉลี่ย     : %.2f%n", sum/count);
        System.out.printf("คะแนนสูงสุด  : %.1f%n", max);
        System.out.printf("คะแนนต่ำสุด  : %.1f%n", min);
        
        // นับเกรด
        int[] gradeCounts = new int[5]; // A,B,C,D,F
        for (int i = 0; i < count; i++) {
            String g = getGrade(scores[i]);
            switch (g) {
                case "A": gradeCounts[0]++; break;
                case "B": gradeCounts[1]++; break;
                case "C": gradeCounts[2]++; break;
                case "D": gradeCounts[3]++; break;
                default:  gradeCounts[4]++;
            }
        }
        
        System.out.println("\nการกระจายเกรด:");
        String[] gradeLabels = {"A", "B", "C", "D", "F"};
        for (int i = 0; i < gradeLabels.length; i++) {
            if (gradeCounts[i] > 0) {
                System.out.printf("เกรด %s: %d คน%n", gradeLabels[i], gradeCounts[i]);
            }
        }
    }
    
    static void searchStudent(String name) {
        boolean found = false;
        for (int i = 0; i < count; i++) {
            if (names[i].equalsIgnoreCase(name)) {
                System.out.printf("พบ: %s คะแนน %.1f เกรด %s%n",
                    names[i], scores[i], getGrade(scores[i]));
                found = true;
            }
        }
        if (!found) System.out.println("ไม่พบ: " + name);
    }
    
    public static void main(String[] args) {
        // เพิ่มข้อมูลตัวอย่าง
        addStudent("สมชาย ใจดี", 85.5);
        addStudent("สมหญิง รักดี", 92.0);
        addStudent("วิชัย เก่งมาก", 67.5);
        addStudent("มาลี สวยงาม", 78.0);
        addStudent("กิตติ ขยัน", 55.0);
        addStudent("นภา ฉลาด", 43.0);
        
        showAll();
        showStats();
        
        System.out.println("\n--- ค้นหา ---");
        searchStudent("สมชาย ใจดี");
        searchStudent("ไม่มีคนนี้");
    }
}
```

---

## 6.8 โปรแกรมตัวอย่าง: Tic-Tac-Toe

```java
import java.util.Scanner;

public class TicTacToe {
    static char[][] board = new char[3][3];
    
    static void initBoard() {
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                board[i][j] = ' ';
    }
    
    static void printBoard() {
        System.out.println("\n  1   2   3");
        System.out.println("┌───┬───┬───┐");
        for (int i = 0; i < 3; i++) {
            System.out.print((i+1) + "│");
            for (int j = 0; j < 3; j++) {
                System.out.print(" " + board[i][j] + " │");
            }
            System.out.println();
            if (i < 2) System.out.println("├───┼───┼───┤");
        }
        System.out.println("└───┴───┴───┘");
    }
    
    static boolean checkWin(char player) {
        // Rows
        for (int i = 0; i < 3; i++)
            if (board[i][0] == player && board[i][1] == player && board[i][2] == player)
                return true;
        // Cols
        for (int j = 0; j < 3; j++)
            if (board[0][j] == player && board[1][j] == player && board[2][j] == player)
                return true;
        // Diagonals
        if (board[0][0] == player && board[1][1] == player && board[2][2] == player)
            return true;
        if (board[0][2] == player && board[1][1] == player && board[2][0] == player)
            return true;
        return false;
    }
    
    static boolean isBoardFull() {
        for (int i = 0; i < 3; i++)
            for (int j = 0; j < 3; j++)
                if (board[i][j] == ' ') return false;
        return true;
    }
    
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        initBoard();
        
        System.out.println("=== Tic-Tac-Toe ===");
        System.out.println("ผู้เล่น 1: X, ผู้เล่น 2: O");
        
        char currentPlayer = 'X';
        
        while (true) {
            printBoard();
            System.out.printf("ผู้เล่น %s - ใส่แถว (1-3): ", currentPlayer);
            int row = sc.nextInt() - 1;
            System.out.printf("ผู้เล่น %s - ใส่คอลัมน์ (1-3): ", currentPlayer);
            int col = sc.nextInt() - 1;
            
            if (row < 0 || row > 2 || col < 0 || col > 2) {
                System.out.println("ตำแหน่งไม่ถูกต้อง!");
                continue;
            }
            
            if (board[row][col] != ' ') {
                System.out.println("ตำแหน่งนี้มีแล้ว!");
                continue;
            }
            
            board[row][col] = currentPlayer;
            
            if (checkWin(currentPlayer)) {
                printBoard();
                System.out.println("ผู้เล่น " + currentPlayer + " ชนะ! 🎉");
                break;
            }
            
            if (isBoardFull()) {
                printBoard();
                System.out.println("เสมอ!");
                break;
            }
            
            currentPlayer = (currentPlayer == 'X') ? 'O' : 'X';
        }
        
        sc.close();
    }
}
```

---

## 6.9 แบบฝึกหัด Part 06

### แบบฝึกหัดที่ 1: Array Statistics

```java
import java.util.Scanner;
import java.util.Arrays;

public class ArrayStats {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        
        System.out.print("จำนวน element: ");
        int n = sc.nextInt();
        
        int[] arr = new int[n];
        System.out.println("ใส่ตัวเลข " + n + " ตัว:");
        for (int i = 0; i < n; i++) {
            System.out.print("  arr[" + i + "] = ");
            arr[i] = sc.nextInt();
        }
        
        // Sort
        int[] sorted = arr.clone();
        Arrays.sort(sorted);
        
        // คำนวณ
        long sum = 0;
        for (int x : arr) sum += x;
        double avg = (double) sum / n;
        
        // Median
        double median;
        if (n % 2 == 0) {
            median = (sorted[n/2 - 1] + sorted[n/2]) / 2.0;
        } else {
            median = sorted[n/2];
        }
        
        System.out.println("\n=== ผลลัพธ์ ===");
        System.out.println("Array: " + Arrays.toString(arr));
        System.out.println("Sorted: " + Arrays.toString(sorted));
        System.out.println("Sum: " + sum);
        System.out.printf("Average: %.2f%n", avg);
        System.out.printf("Median: %.1f%n", median);
        System.out.println("Max: " + sorted[n-1]);
        System.out.println("Min: " + sorted[0]);
        
        sc.close();
    }
}
```

### แบบฝึกหัดที่ 2: Matrix Operations

```java
public class MatrixOps {
    static void printMatrix(int[][] m, String label) {
        System.out.println(label + ":");
        for (int[] row : m) {
            for (int val : row) {
                System.out.printf("%4d", val);
            }
            System.out.println();
        }
    }
    
    static int[][] multiply(int[][] a, int[][] b) {
        int n = a.length;
        int[][] result = new int[n][n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                for (int k = 0; k < n; k++)
                    result[i][j] += a[i][k] * b[k][j];
        return result;
    }
    
    public static void main(String[] args) {
        int[][] a = {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}};
        int[][] b = {{9, 8, 7}, {6, 5, 4}, {3, 2, 1}};
        
        printMatrix(a, "Matrix A");
        printMatrix(b, "Matrix B");
        printMatrix(multiply(a, b), "A × B");
    }
}
```

---

## 6.10 สรุป Part 06

ในบทนี้คุณได้เรียนรู้:

✅ การสร้างและใช้งาน Array พื้นฐาน  
✅ Array operations (sort, search, copy)  
✅ 2D Arrays  
✅ Sorting Algorithms (Bubble, Selection, Insertion)  
✅ Searching Algorithms (Linear, Binary)  
✅ โปรแกรมจริง: Student Management, Tic-Tac-Toe  

---

*[← Part 05: Loops](./part-05-loops.md) | [Part 07: Strings →](./part-07-strings.md)*
