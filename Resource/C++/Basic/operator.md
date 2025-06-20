
```cpp
//연산자 오버로딩
Time Time::operator+(Time& t) {

    Time sum;

    sum.mins = mins + t.mins;

    sum.hours = hours + t.hours;

    sum.hours += sum.mins / 60;

    sum.mins %= 60;

    return sum;

};
```

