

```cpp
class MyClass 
{ 
	private: int secret_data; // private 멤버 
	public: MyClass(int val) : secret_data(val) {} 
	friend void accessSecret(const MyClass& obj); 
	friend std::ostream& operator<<(std::ostream&, Myclass&);
	
};  


void accessSecret(const MyClass& obj) 
{ 
	std::cout << "Friend function accessed secret_data: " 
		      << obj.secret_data 
		      << std::endl; 
} 

std::ostream& std::ostream& operator<<(std::ostream& os, Time& t)
{
	os << t.hours << "시" << t.min << "분" << endl;
	return os;
}


int main() 
{ 
	MyClass myObj(123); 
	accessSecret(myObj);
	//std::cout << myObj.secret_data; 
	// << friend가 아닌 곳에서는 private 접근 불가 
	return 0; 
}```

- **원칙:** 클래스의 `private` 멤버는 기본적으로 **클래스 내부의 멤버 함수만** 접근할 수 있음

-  `friend` 함수는 이 캡슐화 규칙의 **예외**를 만듬. 
- 클래스 안에 `friend`로 선언된 함수를 통하면 해당 클래스의 멤버 함수가 아니어도 `private` 멤버에 직접 접근 가능함