# sgawa.github.io

- [blogs](/blog-list.html)
- [links](/pages/links.html)
- [about](/pages/links.html)

- [telemetry](/pages/telemetry/index.html)

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