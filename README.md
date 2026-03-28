# sgawa.github.io

- [links](/links.html)

```c++
#include <iostream>
using namespace std;

int main() {
    int n;
    cout << "数字を入力: ";
    cin >> n;

    if (n % 2 == 0) {
        cout << n << " は偶数です。" << endl;
    } else {
        cout << n << " は奇数です。" << endl;
    }
    return 0;
}
```