---
title: 基于C语言简易OOP实现
date: 2025-07-17 10:09:13
tags: OOP
categories: C语言
---

最近在倒腾C语言实现类似于OOP的东西，在油管上看到了这样一种实现方法，昨天尝试了一下，现在记录一下

> [Object-Oriented Programming in regular C](https://www.youtube.com/watch?v=e99VxS8ljjY)

最终的主函数长这样，实现了一个非常简陋的String类以及字符串拼接功能，当然，也几乎没有健壮性。这位博主只是简单提供了一种思路。

~~~c
int main(int argc,char *argv[]){
	String *s1;
	String *s2;
	s1 = mkstring("Hello ");
	s2 = mkstring("World");
	$(s1)->concat(s2);
	printfstr(s1);
	
	free(s1);
	free(s2);
	return 0;
}
~~~

这个设计的核心在于全局this指针以及宏定义(虽然全局的this指针不是很安全)。

~~~c
typedef struct s_string String; 
typedef String* (*method)(String*);

typedef struct s_string{
	method concat;
	int8_t length;
	char data[];
}String;
~~~

首先我们需要用结构体模拟一个String类出来，其中包含了concat方法、长度length以及一个char数组(之前在别处见到的另一种实现多态的方法好像用到了接口结构体跟聚合表，我暂时还没太弄明白，等我弄明白了或许会再写个博客出来)。

> 实际上这个 data[]也可以写成`char *data;`,本质上没什么区别

method实际上是一个函数指针，指向一个返回值为**String\***,参数为**String\***的函数，我们需要自己实现这个函数。

接下来我们为这个类实现构造函数以及打印函数，下面是这两个函数的声明。

~~~c
String *mkstring(char*);
void printfstr(const String*);
~~~

printfstr函数没什么好讲的，这里讲一下mkstring函数。

~~~c
String *mkstring(char*str){
	int16_t len;
	int16_t size;
	String *p;
	
	assert(str);
	len = strlen(str);
	assert(len);
	
	size = len +sizeof(String) +1;
	p = (String*)malloc(size);
	assert(p);
	memset(p,0,size);
	
	memcpy(p->data,str,len);
	p->length = len;
	p->concat = concat_;
	return p;
}
~~~

我们可以先忽略掉这些**assert**(断言)，这个函数进行了以下操作:

1. 根据输入参数计算了String对象中length参数的长度并赋值。

2. 根据输入字符数组的长度申请了足够的内存空间，并且使用memcpy函数将字符数组的内容复制进String对象中。
3. 将自己实现的concat_函数与类中的函数指针进行了绑定。
4. 最后返回了一个指向初始化好的String对象的指针。

> 此处需要注意，C语言字符数组以'\0'作为结尾，在这个函数中，通过memset将整个结构体置0时就相当于将类中char数组最后一位置0了，所以不再需要显式的置0。

现在来看一下这个设计最核心的部分，全局this指针

~~~c
typedef void thisptr;
thisptr* _this;
~~~

可以看到我们创建了一个全局this指针，它将始终指向我们正在操作的String对象。

接下来我们来实现这个concat_方法函数

~~~c
String* concat_(String* input) {
	String* current_this = _this; 
	
	char* temp_input_data = (char*)malloc(input->length + 1);
	strcpy(temp_input_data, input->data);
	
	int16_t original_current_this_length = current_this->length; 
	int16_t new_length = original_current_this_length + input->length;
	size_t new_size = sizeof(String) + new_length + 1;
	
	String* reallocated_string = (String*)realloc(current_this, new_size);
	
	if (reallocated_string == NULL) {
		perror("realloc 失败，无法原地扩展字符串");
		free(temp_input_data);
		return NULL;
	}
	
	current_this = reallocated_string; 
	_this = current_this; 
	
	current_this->length = new_length;
	
	memcpy(current_this->data + original_current_this_length, temp_input_data, input->length);
	
	current_this->data[new_length] = '\0'; 

	free(temp_input_data);
	
	return current_this;
}
~~~

我们先忽略掉错误处理部分，这个函数实现了以下的功能

1. **保存输入字符串数据** 函数首先将 `input` 字符串的数据复制到一个临时缓冲区 `temp_input_data` 中。这是为了防止在 `realloc` 失败时，`input->data` 中的数据丢失，或者如果在 `realloc` 后 `input->data` 指向的内存被释放或移动而导致后续操作出错。

2. **计算新字符串长度和所需内存大小** 它计算了连接后的新字符串的总长度 `new_length`（原字符串长度 + 输入字符串长度），并根据这个新长度计算了 `String` 结构体加上字符串数据所需的总内存大小 `new_size`。

3. **重新分配内存** 函数尝试使用 `realloc` 来扩展当前字符串 `_this` 所占用的内存。`realloc` 会尝试在原地扩展内存，如果原地扩展失败，它会分配一块新的内存区域并将原有数据复制过去，然后释放旧的内存区域。

4. **处理内存重新分配失败** 如果 `realloc` 返回 `NULL`，表示内存重新分配失败。此时，函数会打印错误信息，释放之前分配的临时缓冲区，并返回 `NULL`。

5. **更新当前字符串指针和长度** 如果内存重新分配成功，`current_this`（以及全局或成员变量 `_this`）会更新为 `reallocated_string` 返回的新地址。然后，`current_this` 的 `length` 字段会被更新为 `new_length`。

6. **拷贝输入字符串数据** 使用 `memcpy` 将 `temp_input_data`（即 `input` 字符串的数据）拷贝到 `current_this->data` 的末尾，从 `original_current_this_length` 的位置开始。

7. **添加字符串结束符** 在新字符串的末尾（`new_length` 的位置）添加空字符 `\0`，以确保它是一个合法的 C 字符串。

8. **释放临时缓冲区并返回** 最后，释放之前为 `temp_input_data` 分配的内存，并返回更新后的 `current_this` 指针。

这个函数是我修改过的，博主原代码如下:

~~~c
String *concat_(String *input){
    int16_t len;
    int16_t size;
    char *p;
    String *this;
    this = (String*)_this;
    len = this->length + input->length;
    size = len+sizeof(struct s_string)+1;
    p = this->data +this->length;
    this = (String*)realloc(this,size);
    assert(this);
    memcpy(p,input->data,input->length);
    p = this->data +len;
    *p = '0';
    return this;

}
~~~

可能是因为编译环境不同，这种写法在我的编译环境下会导致严重的内存问题。

现在如果我们想要在主函数中实现字符串拼接，需要以下步骤:

1. 初始化s1,s2。
2. this指针指向s1。
3. 调用s1的concat方法，将s2传入。

体现在代码上如下

~~~c
_this = s1;
s1 = s1->concat(s2);
~~~

> 这里会出现`s1 = s1->concat(*)`的写法,是因为在concat函数中进行realloc操作时，会改变s1指针指向的内存，不管是原地扩容还是在新内存空间扩容，在扩容完成后将地址返回给s1就可以保证不出现悬空指针了。

接下来我们可以实现一个操作宏来简化我们的操作

~~~c
#define $(x) _this = (x);(x) = (x)
~~~

这个宏让我们可以以`$(s1)->concat(s2);`的形式直接调用对象中的方法，展开后本质上跟上面的写法是一样的。

> 注意：这种写法实际上是不安全的，我只是将博主的实现方法照抄下来并且进行记录，暂时还没想到怎么才能优化这种写法
>
> 但是有一点显而易见的就是这个全局的this指针是不安全的。
