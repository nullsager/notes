# cpp 零散知识点汇总

## 随机数

std::uniform_int_distribution模板不写默认为int

std::uniform_real_distribution模板不写默认为double

```cpp
std::random_device rd;
std::mt19937 gen(rd());
std::uniform_int_distribution<> dis_int(0, 1000);
for (int i = 0; i < 5; i++) {
  std::cout << dis_int(gen) << '\n';
}
std::uniform_real_distribution<> dis_double(0.0, 1.0);
for (int i = 0; i < 5; i++) {
  std::cout << dis_double(gen) << '\n';
}
std::uniform_real_distribution<float> dis_float(1.0, 2.0);
for (int i = 0; i < 5; i++) {
  std::cout << dis_float(gen) << '\n';
}
std::mt19937_64 gen64(rd());
std::uniform_int_distribution<int64_t> dis64(std::numeric_limits<int64_t>::min(), std::numeric_limits<int64_t>::max());
for (int i = 0; i < 5; i++) {
  std::cout << dis64(gen64) << '\n';
}
```
