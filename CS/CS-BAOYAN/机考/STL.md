## 字符串
### `string_view`

[知乎介绍](https://zhuanlan.zhihu.com/p/691796211)
```cpp
namespace std {
    template<class charT, class traits = std::char_traits<charT>>
    class basic_string_view {
    public:
        // 构造函数
        constexpr basic_string_view() noexcept;
        constexpr basic_string_view(const charT* str);
        constexpr basic_string_view(const charT* str, size_t len);
        
        // 成员函数
        constexpr const charT* data() const noexcept;
        constexpr size_t size() const noexcept;
        constexpr bool empty() const noexcept;
        constexpr charT operator[](size_t pos) const;
        constexpr charT front() const;
        constexpr charT back() const;
        constexpr basic_string_view substr(size_t pos, size_t count = npos) const;
        constexpr int compare(basic_string_view other) const noexcept;
        constexpr size_t find(basic_string_view str, size_t pos = 0) const noexcept;
        // ...
    };
    
    // 类型别名
    using string_view = basic_string_view<char>;
    using wstring_view = basic_string_view<wchar_t>;
    using u16string_view = basic_string_view<char16_t>;
    using u32string_view = basic_string_view<char32_t>;
}
```

此方法是一个模版类，比 `std::string` 更轻量级。
/Users/kold/Library/Mobile Documents/iCloud~md~obsidian/Documents/KoldGoNote/KoldGoNote/lessons/S05/assignments/DLCO-lab2-report. md


## `Count` 方法
```cpp
class Solution {
public:
    int numberOfBeams(vector<string>& bank) {
        int ans = 0, last = 0;
        for (auto& row : bank) {
            int cnt = count(row.begin(), row.end(), '1');
            if (cnt == 0) continue;
            ans += last * cnt;
            last = cnt;
        }
        return ans;
    }
};

```

这里用到了一个 `count` 方法：
`std::count` 是 C++ 标准库 `<algorithm>` 头文件里的一个非常实用的算法，用来**统计某个值在区间中出现的次数**。  

`count_if` 是一个升级的方法，可以加一个函数作为比较条件。


## `priority_queue`
https://cppreference.cn/w/cpp/container/priority_queue
