```cpp
std::vector<std::string> subject{"Data Structures",  "Computer Architecture", "Operating System",
                                 "Computer Network", "Advanced Mathematics",  "Linear Algebra",
                                 "English",          "Political Science"};
std::vector<int> probabilities{5, 15, 15, 15, 10, 10, 25, 5};
if (int sum = std::accumulate(probabilities.begin(), probabilities.end(), 0); sum > 100) {
  std::println("The sum of the probabilities of the options is {}, exceeds 100%.", sum);
  return;
}

const auto& beijingZone = std::chrono::locate_zone("Asia/Shanghai");
const std::chrono::zoned_time localtime{beijingZone, std::chrono::system_clock::now()};
auto now = localtime.get_local_time();
std::string fileName = std::format("{:%m-%d}.txt", now);
std::ofstream file(fileName, std::ios::app);
std::random_device rd;
std::mt19937 gen(rd());

std::discrete_distribution<> dist(probabilities.begin(), probabilities.end());
int idx = dist(gen);
std::println("Please enter the minutes you want to learn {}...", subject[idx]);
int minutes;
std::cin >> minutes;
auto endTime = now + std::chrono::minutes(minutes);
file << std::format("{:%H:%M}-{:%H:%M}, {}\n", now, endTime, subject[idx]);
```