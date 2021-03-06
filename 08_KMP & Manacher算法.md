&emsp;&emsp;KMP算法和Manacher算法都是面试中很多算法的最优解原型，虽然这两个算法自身只解决了两个特定的问题，但是其衍生工具对很多其他题目有非常高的参考价值。

# 1.最长[前缀后缀相等]子序列长度数组next[]
- **基本概念**：针对KMP算法中短字符串str2(arr)生成的特定数组。数组中i位置的数值next[i]只和arr[0\~i-1]元素有关，和arr[i]无关。获取最长的子序列长度，使得前缀与后缀相等。
<div align=center>
<img src="./img/08_img/nextArrayConcept.png" width="512px"/>
</div>

- **next[]数组计算**   
在具体计算时，next\[0\]=-1和next\[1\]=0都是固定值；next\[2\]位置通过前两个元素是否相等进行计算；next\[3\]及以后所有元素均可以通过前向查询进行计算（类似于递归）。
    - 1.假设来到了i位置，需要计算next\[i\]的数值。
    - 2.现在已经有了next\[i-1\]的数值，表示从0到i-2位置上的最长子序列长度。
    - 3.根据cn位置的数值和arr[i-1]的数值相等情况进行分支判断。
<div align=center>
<img src="./img/08_img/nextArrayCalculate.png" width="512px"/>
</div>

```java
static int[] getNextArr(char[] arr){
    /* ==========数组构建与初始化========== */
    if (arr.length == 1){
        return new int [] {-1};
    }
    int[] next = new int[arr.length];
    next[0] = -1; // 人为定义-1
    next[1] = 0;  // 认为定义0
    /* =============变量设计============= */
    int i = 2;   // 若长度大于2，从next[2]开始计算
    int cn = 0;  // 最巧妙的变量，指向和arr[i-1]对比的元素，也表示重复范围为[0~cn-1]
    /* ===========循环求next[]=========== */
    while (i < arr.length){
        if (arr[i-1] == arr[cn]){   //==>[情况1]找到了匹配长度，可以计算next[i]数值，并开启新一轮计算
            next[i] = cn + 1; // (1)将正确数值放入next[i]
            cn++;             // (2)cn此时存储next[i]的值，为i+1元素计算做准备
            i++;              // (3)next[i]完成计算，自增计算i+1位置
            /* 单行运行简化：next[i++] = cn++; */
        } else if (cn > 0){         //==>[情况2]若不相等，且能继续使用next往前跳转。此情况只修改cn，不修改i。
            cn = next[cn];
        } else {                    //==>[情况3]此时cn=0，无法找到任何重复前缀后缀长度，next[i]置0，cn已经自动清零。继续考察i+1。
            next[i++] = 0;
        }
    }
    return next;
}
```

# 2.KMP算法
- **基本概念**：KMP算法是一种改进的字符串匹配算法，由D.E.Knuth，J.H.Morris和V.R.Pratt提出，是一种在Brute-Force算法（最差情况下复杂度O(N*M)）的基础上提出的模式匹配算法。其核心是利用匹配失败后的信息，尽量减少模式串与主串的匹配次数以达到快速匹配的目的。KMP算法的输出为：
    - 若str1中存在str2，返回str1中的起始位置；
    - 若str1中不存在str2，返回-1。
<div align=center>
<img src="./img/08_img/bruteForce.png" width="480px"/>
</div>

- **KMP算法流程**：KMP只是在暴力匹配的基础上进行了加速。通过next数组的构建，利用str2字符串内部的**前后缀相等部分**进行跳跃加速。
    - str1以i为开头开始匹配，直到str1[Y]位置才与str2[X]不一致；
    - next[X]的值指向了最长前缀的下一个位置p；
    - 将str2[p]对齐到str1[Y]，等价于str1从str1[j]为开头进行全串匹配，进行相等判断：若相等则继续匹配，若不等则继续前跳直到p=0或next[p]=-1。
<div align=center>
<img src="./img/08_img/KMPConcept.png" width="512px"/>
</div>
<div align=center>
<img src="./img/08_img/KMPDetails.png" width="512px"/>
</div>

- **KMP跳转合理性分析**：能够跳转且能够继续匹配需要满足以下两点要求
    - 1.str2[X]跳转后str2[p]和str1[Y]匹配，等价于str1[j]从头开始匹配，需要求str1[j==>Y-1]和str2[0==>p-1]位置完全匹配。因为next的内涵可以保证；
    - 2.保证在str1[i]到str1[j]之间没有任何一个字符起始能够匹配出整个str2。此证法需要用到k假设，如下图所示：
<div align=center>
<img src="./img/08_img/KMPJump.png" width="512px"/>
</div>

- **KMP复杂度分析**：循环中共有3个分支，构造变量(i1-i2)，分析i1和(i1-i2)两个变量的变化幅度。两者变化范围最大都是N，因此复杂度为O(N)。
    - 分支1：连续匹配情况，同时推高i1和i2。导致i1升高，(i1-i2)不变；
    - 分支2：无法匹配情况，只推高i1。导致i1升高，(i1-i2)升高；
    - 分支3：回跳情况，只降低i2。导致i1不变，(i1-i2)升高；
<div align=center>
<img src="./img/08_img/KMPComplexity.png" width="320px"/>
</div>

```java
public static int KMP(String s, String m) {
    if (s.length() < m.length() || s == null || m == null || m.length() < 1) {
        return -1;
    }
    char[] str1 = s.toCharArray();
    char[] str2 = m.toCharArray();
    int[] next = getNextArr(str2); // O(M)
    int i1 = 0; // 只增不减
    int i2 = 0; // 可以减小
    // O(N)
    while (i1 < str1.length && i2 < str2.length) { // 检测两个下标是否越界
        if(str1[i1] == str2[i2]) {
            /* ================ 情况1：若一致则一起增长 ================ */
            i1++;
            i2++;
        } else if (i2 == 0) { // 等价于next[i2] == -1
            /* == 情况2：实在无法匹配，i1你往下动一动，从下一个位置开始从零匹配 == */
            i1++; // 现在i2已经清零，无法再往下跳了
        } else {
            /* ========== 情况3：两者不相等，i2可以根据next回跳 ========== */
            i2 = next[i2];
        }
    }
    // 出现越界情况，若i2满位则说明发生了匹配，返回i1-12；若i2没满位则说明未匹配上
    return i2 == str2.length ? i1 - i2 : -1;
}

public static int[] getNextArr(char[] arr) {
    if (arr.length == 1) {
        return new int[]{-1};
    }
    int[] next = new int[arr.length];
    next[0] = -1;
    next[1] = 0;
    int i = 2;
    int cn = 0;
    while (i < arr.length) {
        if(arr[i-1] == arr[cn]) {
            next[i++] = ++cn;
        } else if (cn > 0) {
            cn = next[cn];
        } else {
            next[i++] = 0;
        }
    }
    return next;
}

public static void main(String[] args) {
    String s1 = "abc123";
    String s2 = "123";
    int out = KMP(s1, s2);
    System.out.println(out);
}
```

# 3.最长回文半径数组rArr[]
- **基本概念**：字符串中每个字符位置的`最长回文子序列半径`数组，能够应用在很多字符串问题求解中。注意半径r囊括中心点。
- **字符串预处理**：按照直觉的想法，以每一个位置为中心进行两边扩展的暴力寻找策略无法处理`以虚轴为中心的回文串`，因此需要进行字符串预处理，添加补充字符`#`（可以是任意字符）。在最差情况下，例如`#1#1#1#1#1#1#1#`，每个位置的寻找过程总会扩展到左右边界，导致整体的复杂度为O(N^2)。
<div align=center>
<img src="./img/08_img/ManacherRArray.png" width="440px"/>
</div>

# 4.Manacher算法
- **基本概念**：解决单个字符串中`最长回文子串`长度求解问题，能够将搜索时间控制在O(N)级别。注意Manacher算法能解决回文串的问题但不仅于此，`回文半径数组`的信息可以用于解决好多回文问题，解决最长回文子串只是其一种功效而已。

- **重要数组与变量**：
    - **1.最长回文半径数组rArr[]**：存储每个位置向左右扩张的最大半径。
    - **2.最右回文右边界R**：之前扩的所有位置中，所到达的最右回文右边界位置。此值随着考察点i的右移而右移。
    - **3.取得最右边界时的中心C**：与R是伴生同步更新的，R记录最远疆界，C记录此疆界对应的中心。
<div align=center>
<img src="./img/08_img/ManacherR&C.png" width="360px"/>
</div>

- **Manacher算法的情况罗列**：
    - **1.当i位置出现在R外时**：i已经位于R右侧(i没有在最远回文右边界里)，进行双侧暴力扩，并且一定能够更新R和C。即`--[L---C---R]i--`；
    - **2.当i位置出现在R内时**：拓扑关系一定是`--[L--j---C---i--R]--`，其中j为i关于C的对称点。现在根据j区域的回文状况进行详细分类：
        - **(1).j的回文区域严格在(L-C)内部**：`--[L-{-j-}--C---i--R]--`，此时可以直接断定i位置的回文区域和j回文区域完全一致：`--[L-{-j-}--C--{-i-}-R]--`。可以根据i回文串左右两元素必定不相同进行推理。
        - **(2).j的回文区域跑到了[L-C]外侧**：`-{-[L--j---}-C----i--R]--`，此时可以直接断定i位置的回文区域最右边界为R：`-{-[L--j---}-C--{--i--R}]--`。可以根据i回文串左右两元素必定不相同进行推理。
        - **(3).j的回文区域和L刚好压线**：`--[{L---j---}--C-----i---R]--`，此时可以确保i位置到R位置一定是回文区域，但是是否外扩还需要判断：`--[{L---j---}--C-X{---i---R}]?-`，若X=?则能够进行扩展并更新右边界R和中心C，若X!=?则扩展失败并且右边界不会被改变。（此位置出现了一个小加速过程，因为已经确保`{---i---R}`一定是回文，所以直接从?位置开始检验）
<div align=center>
<img src="./img/08_img/ManacherCase.png" width="560px"/>
</div>

- **Manacher时间复杂度分析**：从扩展的失败和成功两个角度分析复杂度，共分为两种情况。
    - **失败情况**：每个位置最多扩展失败1次，只有大情况1与情况2.3会导致扩张失败，因此扩张失败复杂度为O(N)；
    - **成功情况**：从i和R的变化幅度进行分析，大情况1同时导致i和R上升、情况2.1和2.2直接出结果所以只导致i上升、情况2.3同时导致i和R上升。i与R两者均是只增不退，最大变化幅度为N，因此成功扩张次数也为O(N);
    - **总结**：整体四个分支是互斥关系；成功扩张情况下每一次扩都会使得R变大，R变大的总幅度其实就是扩成功的次数O(N)（外扩行为和R变大行为绑定在一起）；失败扩张次数可以统计O(N)；因此总体复杂度O(N)。
<div align=center>
<img src="./img/08_img/ManacherComplexity.png" width="240px"/>
</div>

```java
/* Manacher伪代码，用于分析时间复杂度 */
public static int[] manacher(String s){
    char[] str = f(s); // 预处理添加#

    int[] rArr = new int[str.length]; // 回文半径数组
    int R = -1; // 最右回文右边界R
    int C = -1; // 取得最右边界时的中心C

    for (int i = 0; i < str.length; i++){
        if (i在R右侧){ // --[L---C---R]i--
            从i开始左右暴力扩展;
            一定能够更新R和C;
        } else{
            if (j的回文彻底在L-R内){                // --[L-{-j-}--C---i--R]--
                rArr[i] = O(1)表达式; // 直接锁定i区域
            } else if (j的回文区域跑到了L-C外侧){   // -{-[L--j---}-C----i--R]--
                rArr[i] = O(1)表达式; // 直接锁定i区域
            } else{// j的回文区域和L刚好压线        // --[{L---j---}--C-----i---R]?-
                从R之外的字符开始往外扩，确定rArr[i]数值、边界R和中心C;
                若能扩:R和C变大
                若不能扩:R和C保持不变
            }
        }
    }
}
```

- **Manacher基本实现**：按照两种大情况和三种小情况进行分类实现，代码较长但是逻辑结构清晰。注意其中的R使用的是闭区间表示形式，R指向的就是最长回文序列到达的最右侧元素。
```java
public static char[] getManacherStr(String s) {
    char[] str = s.toCharArray();
    char[] strPad = new char[2 * str.length + 1]; // 2*L+1
    strPad[0] = '#';
    for (int i = 1; i < strPad.length; i += 2) {
        strPad[i] = str[i / 2];
        strPad[i + 1] = '#';
    }
    return strPad;
}

public static int Manacher(String s) {
    if (s == null) {
        return -1;
    }
    char[] str = getManacherStr(s);
    int[] rArr = new int[str.length]; // 最长回文半径数组
    int R = -1; // 最远回文右边界，使用闭区间表示方法
    int C = -1; // 和R伴生，中心位置
    int curMax = Integer.MIN_VALUE;

    for (int i = 0; i < str.length; i++) { // 指针定为i
        /* ====== 大情况1:i出现在R外 ====== */
        if (R < i) {
            int p = 0;
            while ((i - p) >= 0 && (i + p) < str.length) {
                if (str[i - p] == str[i + p]) {
                    rArr[i]++;
                    p++; // 此时p的值指向了一个非法值，例如p=0时指向i自身，但是出循环变为1，需要进行减一操作
                } else {
                    break;
                }
            }
            C = i;
            R = i + (p - 1);
        } else {
            int j = 2 * C - i; // 对称点
            int L = 2 * C - R; // 左边界（闭区间）
            /* ============== 情况1:在R内，直接出结果 ============== */
            if (L < (j - rArr[j] + 1)) { // j位置-(回文半径-1) = j-(rArr[j]-1)才能够锁定[--j--]的左边界位置，进而和L进行判断
                rArr[i] = rArr[j];
            /* ============== 情况2:从i到R，直接出结果 ============== */
            } else if ((j - rArr[j] + 1) < L) {
                rArr[i] = R - i + 1;
            /* ============== 情况3:压线情况，进行判断性外扩 ============== */
            } else if ((j - rArr[j] + 1) == L) {
                int p = 0;
                while ((i - p) >= 0 && (i + p) < str.length) {
                    if (str[i - p] == str[i + p]) {
                        rArr[i]++;
                        p++;
                    } else {
                        break;
                    }
                }
                if (i + p - 1 > R) { // 出现外扩情况，更新R和C
                    C = i;
                    R = i + (p - 1);
                }
            }
        }
        if (rArr[i] > curMax) { // 填充之后的字符串，最终回文长度为最长半径-1：#1#2#[3#2#1#] ==> 5
            curMax = rArr[i];
        }
    }
    return curMax - 1;
}

public static void main(String[] args) {
    String s1 = "abc12321cba3";
    System.out.println(Manacher(s1));
}
```

- **Manacher简洁实现**：对四种互斥情况进行整合，先完成最小不需要检验区域的锁定，再统一进行扩张，不会对复杂度造成任何影响。注意此时仍然使用闭区间表示方法。
```java
public static int ManacherPro(String s) {
    if (s == null || s.length() == 0) {
        return -1;
    }
    char[] str = getManacherStr(s);
    int[] rArr = new int[str.length]; // 最长回文半径数组
    int R = -1; // 最远回文右边界，使用闭区间表示方法
    int C = -1; // 和R伴生，中心位置
    int curMax = Integer.MIN_VALUE;

    for (int i = 0; i < str.length; i++) {
        /* ================ 获取最小不需要检验区域，绝了！ ================ */
        rArr[i] = R < i ? 1 : Math.min(rArr[2 * C - i], R - i + 1);
        /* ============== 基于最小不需要检验区域进行暴力外扩 =============== */
        while (i + rArr[i] < rArr.length && i - rArr[i] > -1) {
            if (str[i + rArr[i]] == str[i - rArr[i]]) { // 保证大情况1和情况2.3能够进行外扩
                rArr[i]++;
            } else { // 保证情况2.1和2.2都只扩张一次失败
                break;
            }
        }
        if (i + rArr[i] - 1 > R) { // 若i位置的半径拓宽了R，则进行R和C的更新
            C = i;
            R = i + rArr[i] - 1;
        }
        if (curMax < rArr[i]) { // 更新最长半径值
            curMax = rArr[i];
        }
    }
    return curMax - 1;
}
```
