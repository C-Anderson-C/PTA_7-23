# PTA_7-23
# 币值转换
# 输入一个整数（位数不超过9位）代表一个人民币值（单位为元），请转换成财务要求的大写中文格式。如23108元，转换后变成“贰万叁仟壹百零捌”元。为了简化输出，用小写英文字母a-j顺序代表大写数字0-9，用S、B、Q、W、Y分别代表拾、百、仟、万、亿。于是23108元应被转换输出为“cWdQbBai”元。

# 输入格式：输入在一行中给出一个不超过9位的非负整数。

# 输出格式：在一行中输出转换后的结果。注意“零”的用法必须符合中文习惯

```cpp
#include <iostream>
#include <string>
using namespace std;

// 将数字0-9分别对应 a-j（a=0, b=1, ... j=9）
string digits = "abcdefghij";

// 根据组内位数确定各位中文单位
// 对于4位组，依次对应仟、百、拾、空；对于3位组，对应百、拾、空；对于2位组对应拾、空；1位组无单位。
// 注意：转换时只有非0数字才输出其后单位，如果前面有零则只保留一个"a"表示零。
string convertGroup(const string &group) {
    int n = group.size();
    string result;
    bool zeroFlag = false;
    string unit[4]; //最多用到4个单位
    if(n == 4) {
        unit[0] = "Q"; // 仟
        unit[1] = "B"; // 百
        unit[2] = "S"; // 拾
        unit[3] = "";  // 个位无单位
    } else if(n == 3) {
        unit[0] = "B"; // 百
        unit[1] = "S"; // 拾
        unit[2] = "";
    } else if(n == 2) {
        unit[0] = "S"; // 拾
        unit[1] = "";
    } else { // n == 1
        unit[0] = "";
    }
    
    // 从左到右扫描
    for (int i = 0; i < n; i++) {
        int d = group[i] - '0';
        if(d == 0) {
            // 置零标记——连续多个零只记录一次，实际输出推迟到遇到非零时输出"a"
            zeroFlag = true;
        } else {
            // 如果前面有零，先补一个"a"
            if(zeroFlag) {
                result += "a";
                zeroFlag = false;
            }
            result.push_back(digits[d]);  // 输出数字对应字母
            // 非最后一位才输出单位
            if(i < n - 1) {
                result += unit[i];
            }
        }
    }
    return result;
}

// 主转换函数：将整数转换成中文大写（简化形式）
// 处理亿、万、个三个部分，注意：若万组（或亿组）全零，但后面低位非零时要保留一个零"a"
string convertToChinese(int n) {
    if(n == 0)
        return "a";
    
    string numStr = to_string(n);
    int len = numStr.size();
    
    // 分组：根据数字位数，从右侧开始，每4位作为一组
    string g1, g2, g3; // 分别代表个位（低4位）、万组（中间最多4位）、亿组（剩余部分）
    if(len <= 4) {
        g1 = numStr;
    } else if(len <= 8) {
        int split = len - 4;
        g2 = numStr.substr(0, split);
        g1 = numStr.substr(split);
    } else {  // len从9位开始（本题不超过9位，所以最多只有1位亿组）
        int split1 = len - 8;
        g3 = numStr.substr(0, split1);
        g2 = numStr.substr(split1, 4);
        g1 = numStr.substr(split1 + 4);
    }
    
    string result;
    bool needZero = false; // 标记万组全零但低组非零时需要插入“零”
    
    // 先转换亿组（最高组），若不为空且转换后非空则加上“亿”（用"Y"表示）
    if(!g3.empty()){
        string part = convertGroup(g3);
        if(!part.empty()){
            result += part + "Y";
        }
    }
    
    // 转换万组
    if(!g2.empty()){
        string part = convertGroup(g2);
        if(!part.empty()){
            result += part + "W";
        } else { // 万组存在但全是0
            needZero = true;
        }
    }
    
    // 转换个位组（低4位）
    if(!g1.empty()){
        string part = convertGroup(g1);
        if(!part.empty()){
            // 如果上级组万组为空（但存在）时且低组不为空则在前面补一个零
            if(needZero && !result.empty() && result.back() != 'a')
                result += "a";
            result += part;
        }
    }
    
    return result;
}

int main() {
    int n;
    cin >> n;
    // 输出转换后的结果，并在末尾加上"元"
    cout << convertToChinese(n) << endl;
    return 0;
}
