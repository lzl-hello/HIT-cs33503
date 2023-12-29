



# 哈尔滨工业大学CS33503数据库系统实验报告

姓名：李卓凌

学号：2021111000

班级：2103201

专业：信息安全

学期：2023秋

## 声明

本人承诺该实验全部由本人独立完成，没有抄袭他人代码。若被证实本人存在抄袭现象或为他人抄袭提供帮助，本人愿意承担全部责任（包括但不限于扣分、取消考试资格、上报学部等）。

## 目录

[TOC]

## 实验1：存储管理实验


### 任务1.1：磁盘存储管理器

#### DiskManager类的实现

1. `void DiskManager::write_page(int fd, page_id_t page_no, const char *offset, int num_bytes);`

**方法声明：**
```c++
void write_page(int fd, page_id_t page_no, const char *offset, int num_bytes);
```

**方法实现思路：**
- 使用`lseek()`函数将文件描述符(`fd`)定位到文件头，通过`(page_no * PAGE_SIZE)`可以定位到指定页面及其在磁盘文件中的偏移量。
- 调用`write()`函数将指定字节数(`num_bytes`)从内存中的`offset`位置写入到磁盘文件中的指定页面位置。

**方法实现难点：**
- 主要难点在于理解文件描述符和页面号之间的映射关系，确保在磁盘上正确定位到指定页面。

2. `void DiskManager::read_page(int fd, page_id_t page_no, char *offset, int num_bytes);`

**方法声明：**
```c++
void read_page(int fd, page_id_t page_no, char *offset, int num_bytes);
```

**方法实现思路：**
- 与`write_page`功能类似，只是变为调用`read`函数。
- 使用`lseek()`函数将文件描述符(`fd`)定位到文件头，通过`(page_no * PAGE_SIZE)`可以定位到指定页面及其在磁盘文件中的偏移量。
- 调用`read()`函数将磁盘文件中指定页面的内容读取到内存的指定位置(`offset`)。

**方法实现难点：**
- 同样的映射关系问题，确保正确读取磁盘上指定页面的内容。

3. `page_id_t DiskManager::AllocatePage(int fd);`

**方法声明：**
```c++
page_id_t AllocatePage(int fd);
```

**方法实现思路：**
- 简单的自增分配策略，指定文件的页面编号加1。
- 先获取当前文件的页面编号，再调用函数+1，最后返回该页面编号。

**方法实现难点：**
- 无明显难点，主要考察对页面编号的管理和分配策略的理解。

4. `bool is_file(const std::string &path);`

**方法声明：**
```c++
bool is_file(const std::string &path);
```

**方法实现思路：**
- 使用`struct stat`获取文件信息，在`sys/stat.h`中定义。
- 使用`stat(path.c_str(), &st)`函数获取指定路径的文件信息，将结果存储在`st`中。`stat`函数返回0表示成功，-1表示失败。
- 返回一个布尔值，表示给定路径是否表示一个普通文件。返回值条件为`stat`函数返回值为0（表示成功）并且`S_ISREG(st.st_mode)`为真，即该路径表示一个普通文件。

**方法实现难点：**
- 需要理解`struct stat`的使用，以及文件类型的判断条件。

5. `void create_file(const std::string &path);`

**方法声明：**
```c++
void create_file(const std::string &path);
```

**方法实现思路：**
- 检查文件是否已存在，如果存在则抛出`FileExistsError`异常。
- 调用`open()`函数创建文件，使用`O_CREAT|O_RDWR`标志，并设置文件权限为`0600`。
- 使用`O_CREAT`标志表示如果文件不存在则创建，`O_RDWR`表示以可读写方式打开，同时设置文件权限为`0600`。

**方法实现难点：**
- 需要处理文件存在的情况，并理解`open()`函数的使用及其参数。

6. `void open_file(const std::string &path);`

**方法声明：**
```c++
void open_file(const std::string &path);
```

**方法实现思路：**
- 检查文件是否存在，如果不存在则抛出`FileNotFoundError`异常。
- 检查文件是否已打开，进入`if`说明打开文件列表里面没有。
- 使用`open()`函数打开文件，更新文件打开列表。

**方法实现难点：**
- 处理文件不存在和文件已打开的情况，理解`open()`函数的使用。

7. `void close_file(int fd);`

**方法声明：**
```c++
void close_file(int fd);
```

**方法实现思路：**
- 进入`if`说明打开文件列表里面有。
- 获取文件路径，检查文件是否存在。
- 关闭文件，更新文件打开列表。

**方法实现难点：**
- 处理文件不存在和文件已关闭的情况，理解`close()`函数的使用。

8. `void destroy_file(const std::string &path);`

**方法声明：**
```c++
void destroy_file(const std::string &path);
```

**方法实现思路：**
- 检查文件是否存在，如果不存在则抛出`FileNotFoundError`异常。
- 检查文件是否打开，检查文件是否在打开文件列表 (`path2fd_`) 里面。
- 如果文件在打开文件列表中，先关闭文件，然后删除文件。

**方法实现难点：**
- 处理文件不存在、文件已打开和删除文件的情况，理解`unlink()`函数的使用。

9. `int DiskManager::GetFileSize(const std::string &file_name);`

**方法声明：**
```c++
int GetFileSize(const std::string &file_name);
```

**方法实现思路：**
- 使用`struct stat`获取文件信息。
- 调用`stat(file_name.c_str(), &stat_buf)`函数获取指定路径的文件信息，将结果存储在`stat_buf`中。
- 返回文件大小，若文件不存在则返回-1。

**方法实现难点：**
- 理解`struct stat`的使用，处理文件不存在的情况。

10. `std::string DiskManager::GetFileName(int fd);`

**方法声明：**
```c++
std::string GetFileName(int fd);
```

**方法实现思路：**


- 检查文件是否已打开，如果未打开则抛出`FileNotOpenError`异常。
- 返回文件路径。

**方法实现难点：**
- 处理文件未打开的情况，理解文件路径的映射关系。

11. `int DiskManager::GetFileFd(const std::string &file_name);`

**方法声明：**
```c++
int GetFileFd(const std::string &file_name);
```

**方法实现思路：**
- 检查文件是否已打开，如果已打开则返回文件描述符。
- 如果文件未打开，则调用`open_file`打开文件，并返回文件描述符。

**方法实现难点：**
- 处理文件未打开的情况，理解文件描述符的映射关系。

12. `bool DiskManager::ReadLog(char *log_data, int size, int offset, int prev_log_end);`

**方法声明：**
```c++
bool ReadLog(char *log_data, int size, int offset, int prev_log_end);
```

**方法实现思路：**
- 从指定偏移量(`offset + prev_log_end`)读取日志文件的内容。
- 如果日志文件未打开，则使用`open_file`打开日志文件。
- 根据偏移量和日志文件大小，确定读取的字节数。
- 使用`lseek()`函数定位到指定偏移量，然后调用`read()`函数读取日志数据。

**方法实现难点：**
- 确定读取的字节数，处理文件未打开的情况，理解文件偏移量的计算。

13. `void DiskManager::WriteLog(char *log_data, int size);`

**方法声明：**
```c++
void WriteLog(char *log_data, int size);
```

**方法实现思路：**
- 如果日志文件未打开，则使用`open_file`打开日志文件。
- 使用`lseek()`函数定位到文件末尾，然后调用`write()`函数将日志数据写入文件。

**方法实现难点：**
- 处理文件未打开的情况，理解文件末尾的定位和写入数据的操作。



#### 实验发现


1. **`write_page` 方法**
   - **理论知识点**: 磁盘文件中页面的定位与写入。
   - **实现细节**: 使用 `lseek()` 定位到文件中的特定页面，并使用 `write()` 函数将数据写入此位置。
   - **理论对应**: 此方法体现了磁盘存储管理中文件与页面的映射关系理解和页面写入的操作。

2. **`read_page` 方法**
   - **理论知识点**: 读取磁盘文件中的指定页面。
   - **实现细节**: 类似于 `write_page`，但使用 `read()` 读取页面内容。
   - **理论对应**: 体现了页面读取操作，符合磁盘存储中页面读取的理论知识。

3. **`AllocatePage` 方法**
   - **理论知识点**: 页面分配策略。
   - **实现细节**: 采用简单自增策略分配页面编号。
   - **理论对应**: 展示了对页面编号管理和分配策略的理解。

4. **`is_file` 方法**
   - **理论知识点**: 文件系统中文件的识别与类型判断。
   - **实现细节**: 使用 `struct stat` 和 `stat()` 函数判断路径是否代表一个普通文件。
   - **理论对应**: 对文件类型的判断，相关于文件系统中文件类型的识别。

5. **`create_file` 方法**
   - **理论知识点**: 文件的创建与权限设置。
   - **实现细节**: 使用 `open()` 函数创建文件，并设置适当的权限。
   - **理论对应**: 体现了文件创建和权限管理的理论知识。

6. **`open_file` 方法**
   - **理论知识点**: 文件的打开与文件描述符管理。
   - **实现细节**: 检查文件是否存在，然后使用 `open()` 打开文件，并更新文件打开列表。
   - **理论对应**: 展示了文件打开操作和文件描述符的管理。

7. **`close_file` 方法**
   - **理论知识点**: 文件的关闭和资源管理。
   - **实现细节**: 使用 `close()` 函数关闭文件，并更新文件打开列表。
   - **理论对应**: 对文件关闭操作和资源管理的理论知识的应用。


### 任务1.2：缓冲池替换策略

#### LRUReplacer类的实现

1. `Replacer(size_t num_pages);`

**方法声明：**
```c++
Replacer(size_t num_pages);
```

**方法实现思路：**
- 构造函数，初始化LRUlist的最大容量`max_size`。在构造对象时，为LRUReplacer指定最大容量，以确定LRUlist的大小。

**方法实现难点：**
- 无特殊难点，主要是参数的正确传递和对象成员变量的初始化。

2. `bool Victim(frame_id_t *frame_id);`

**方法声明：**
```c++
bool Victim(frame_id_t *frame_id);
```

**方法实现思路：**
- 使用`std::scoped_lock`进行上锁操作，确保线程安全。
- 检查LRUlist是否为空，如果为空表示没有可淘汰的页面，返回`false`。
- 从LRUlist末尾取出最近最少使用的帧号，将帧号传递给`frame_id`。
- 从LRUlist中移除该帧。

**方法实现难点：**
- 处理LRUlist为空的情况，理解LRU替换策略的核心思想。

3. `void Pin(frame_id_t frame_id);`

**方法声明：**
```c++
void Pin(frame_id_t frame_id);
```

**方法实现思路：**
- 使用`std::scoped_lock`进行上锁操作，确保线程安全。
- 遍历LRUlist，找到指定的帧并从LRUlist中移除。如果未找到指定帧，则函数不执行任何操作。

**方法实现难点：**
- 处理未找到指定帧的情况，确保正确处理固定帧的操作。

4. `void Unpin(frame_id_t frame_id);`

**方法声明：**
```c++
void Unpin(frame_id_t frame_id);
```

**方法实现思路：**
- 使用`std::scoped_lock`进行上锁操作，确保线程安全。
- 遍历LRUlist，如果在LRUlist中找到指定帧，则无需进行任何操作。
- 如果未找到指定帧，将其添加到LRUlist的开头，表示取消固定并重新参与替换策略。

**方法实现难点：**
- 处理在LRUlist中找到和未找到指定帧的两种情况，理解取消固定帧后重新参与替换策略的逻辑。

5. `size_t LRUReplacer::Size();`

**方法声明：**
```c++
size_t LRUReplacer::Size();
```

**方法实现思路：**
- 直接返回LRUlist的大小。

**方法实现难点：**
- 无特殊难点，理解返回LRUlist大小的操作。




#### 实验发现


1. **`Replacer(size_t num_pages)` 构造函数**
   - **理论知识点**: LRU替换策略的初始化和参数配置。
   - **实现细节**: 初始化LRU列表的最大容量 `max_size`。
   - **理论对应**: 体现了LRU替换策略的基本设置和初始化过程。

2. **`bool Victim(frame_id_t *frame_id)` 方法**
   - **理论知识点**: LRU替换策略的核心操作，选择最近最少使用的页面。
   - **实现细节**: 使用锁保证线程安全，从LRU列表末尾选择最近最少使用的帧。
   - **理论对应**: 体现了LRU替换策略的核心原理，即“最近最少使用”的页面选择机制。

3. **`void Pin(frame_id_t frame_id)` 方法**
   - **理论知识点**: 对特定页面的固定操作，防止其被替换。
   - **实现细节**: 遍历LRU列表，找到并移除指定帧。
   - **理论对应**: 展示了如何在LRU替换策略中处理固定页面。

4. **`void Unpin(frame_id_t frame_id)` 方法**
   - **理论知识点**: 对特定页面取消固定，使其再次参与替换策略。
   - **实现细节**: 若指定帧不在LRU列表中，将其添加至列表。
   - **理论对应**: 体现了管理缓冲池中页面状态的方法，尤其是取消固定后重新参与替换的过程。

5. **`size_t LRUReplacer::Size()` 方法**
   - **理论知识点**: LRU列表的大小管理。
   - **实现细节**: 直接返回LRU列表的当前大小。
   - **理论对应**: 体现了对缓冲池中管理的页面数量的监控和理解。



### 任务1.3：缓冲池管理器

#### BufferPoolManager类的实现

1. `bool FindVictimPage(frame_id_t *frame_id);`

**方法声明：**
```c++
bool FindVictimPage(frame_id_t *frame_id);
```

**方法实现思路：**
- 用于寻找淘汰页。实现逻辑包括从`free_list`或`replacer`中选择可淘汰的帧，返回成功找到的可替换帧id。

**方法实现难点：**
- 实现时需要考虑缓冲池是否已满，选择合适的替换策略。

2. `void UpdatePage(Page *page, PageId new_page_id, frame_id_t new_frame_id);`

**方法声明：**
```c++
void UpdatePage(Page *page, PageId new_page_id, frame_id_t new_frame_id);
```

**方法实现思路：**
- 用于更新页表和页面。根据传入的`Page`指针、新的`PageId`和新的帧号，更新页表和页面信息。

**方法实现难点：**
- 在更新页面时需要考虑页面的元数据，如数据、脏页标志等的更新。

3. `BufferPoolManager(size_t pool_size, DiskManager *disk_manager);`

**方法声明：**
```c++
BufferPoolManager(size_t pool_size, DiskManager *disk_manager);
```

**方法实现思路：**
- 构造函数，需要初始化缓冲池的最大容量`pool_size`，以及分配`replacer`和`pages`的地址空间。初始时，`free_list`中帧编号的范围为`[0, pool_size)`。

**方法实现难点：**
- 在构造函数中需要正确初始化缓冲池的相关参数，包括`pool_size`、`disk_manager`、`replacer`、`pages`等。

4. `Page *NewPage(PageId *page_id);`

**方法声明：**
```c++
Page *NewPage(PageId *page_id);
```

**方法实现思路：**
- 用于在内存申请创建一个新的页面。实现逻辑包括更新页表和页面、固定页面、寻找淘汰页等。使用`DiskManager`分配页面编号，并传出这个新页面的编号。

**方法实现难点：**
- 在新建页面时需要考虑更新页表、页面的固定和淘汰策略。

5. `Page *FetchPage(PageId page_id);`

**方法声明：**
```c++
Page *FetchPage(PageId page_id);
```

**方法实现思路：**
- 用于获取缓冲池中的指定页面。实现逻辑包括更新页表和页面、固定页面、寻找淘汰页等。如果缓冲池中不存在该页面，需要使用`DiskManager`从磁盘中读取。

**方法实现难点：**
- 在获取页面时需要考虑页面的固定和淘汰策略，以及是否需要从磁盘中读取。

6. `bool UnpinPage(PageId page_id, bool is_dirty);`

**方法声明：**
```c++
bool UnpinPage(PageId page_id, bool is_dirty);
```

**方法实现思路：**
- 用于使用完页面后，对该页面取消固定。实现逻辑包括减少页面的一次引用次数，当引用次数减少到0时，调用`Replacer::Unpin()`来取消固定页面所在的帧。参数`is_dirty`决定是否对页面置脏。

**方法实现难点：**
- 在取消固定页面时需要考虑引用次数、脏页标志和是否需要调用`Replacer::Unpin()`。

7. `bool DeletePage(PageId page_id);`

**方法声明：**
```c++
bool DeletePage(PageId page_id);
```

**方法实现思路：**
- 用于删除指定页面。实现逻辑包括更新页表和页面、更新空闲帧列表等。只有引用次数为0的页面才能被删除。

**方法实现难点：**
- 在删除页面时需要考虑引用次数、空闲帧列表等。

8. `bool FlushPage(PageId page_id);`

**方法声明：**
```c++
bool FlushPage(PageId page_id);
```

**方法实现思路：**
- 用于强制刷新（写入）缓冲池中的指定页面到磁盘。"强制"指的是无论该页的引用次数是否大于0，无论该页是否为脏页，都将其刷新到磁盘。

9. `void FlushAllPages(int fd);`

**方法声明：**
```c++
void FlushAllPages(int fd);
```

**方法实现思路：**
- 用于将指定文件中的存在于缓冲池的所有页面都刷新到磁盘。在上述所有函数的实现中，淘汰脏页之前，都要将脏页写入磁盘。

#### 实验发现


1. **`FindVictimPage(frame_id_t *frame_id)` 方法**
   - **理论知识点**: 缓冲池中页面的替换策略。
   - **实现细节**: 从`free_list`或`replacer`中选择可替换的帧。
   - **理论对应**: 展示了缓冲池中如何选择淘汰页面，体现了页面替换策略的理论应用。

2. **`UpdatePage(Page *page, PageId new_page_id, frame_id_t new_frame_id)` 方法**
   - **理论知识点**: 页面信息的更新。
   - **实现细节**: 更新页表和页面信息，包括页面的元数据。
   - **理论对应**: 体现了缓冲池中对页面信息管理和更新的操作，符合页面管理的理论知识。

3. **`BufferPoolManager(size_t pool_size, DiskManager *disk_manager)` 构造函数**
   - **理论知识点**: 缓冲池的初始化和配置。
   - **实现细节**: 初始化缓冲池的容量、`replacer`和`pages`的地址空间。
   - **理论对应**: 体现了缓冲池配置和初始化的理论知识。

4. **`NewPage(PageId *page_id)` 方法**
   - **理论知识点**: 新页面的创建和管理。
   - **实现细节**: 在内存中创建新页面，涉及页表更新、页面固定和淘汰策略。
   - **理论对应**: 体现了缓冲池管理中新页面创建和管理的理论知识。

5. **`FetchPage(PageId page_id)` 方法**
   - **理论知识点**: 获取缓冲池中的指定页面。
   - **实现细节**: 获取指定页面，涉及页面固定、淘汰策略，以及可能的磁盘读取操作。
   - **理论对应**: 展示了缓冲池中页面获取的过程，包括页面状态管理和磁盘读取操作。

6. **`UnpinPage(PageId page_id, bool is_dirty)` 方法**
   - **理论知识点**: 页面的取消固定和状态更新。
   - **实现细节**: 未提供详细实现细节，但一般包括取消页面的固定状态和更新页面的脏页标志。
   - **理论对应**: 体现了缓冲池中管理页面状态的方法，特别是关于页面固定状态的管理和脏页标志的更新。


### 任务2.1: 记录操作

#### RMFileHandle类的实现

1. `RmPageHandle create_new_page_handle();`

**方法声明：**
```c++
RmPageHandle create_new_page_handle();
```

**方法实现思路：**
- 利用缓冲池创建新页，更新`page_hdr`和`file_hdr`中的各项内容。
- 内部实现逻辑包括使用`BufferPoolManager`创建新的页面，更新该页面的`page_hdr`信息，同时更新文件的`file_hdr`信息。

**方法实现难点：**
- 主要难点在于正确地创建新页面，管理`page_hdr`和`file_hdr`的信息，以及合理地使用缓冲池。


2. `RmPageHandle fetch_page_handle(int page_no) const;`

**方法声明：**
```c++
RmPageHandle fetch_page_handle(int page_no) const;
```

**方法实现思路：**
- 用于获取指定页面对应的`RmPageHandle`。
- 内部实现逻辑包括使用`BufferPoolManager`从缓冲池中获取指定页面的帧，并将其封装成`RmPageHandle`返回。

**方法实现难点：**
- 主要难点在于正确地使用缓冲池管理器，确保获取到的页面是正确的。


3. `RmPageHandle create_page_handle();`

**方法声明：**
```c++
RmPageHandle create_page_handle();
```

**方法实现思路：**
- 用于创建或获取一个空闲的`RmPageHandle`。
- 内部实现逻辑是先判断第一个空闲页是否存在，如果存在就直接用`fetch_page_handle()`获取它；否则直接用`create_new_page_handle()`创建一个新的`RmPageHandle`。

**方法实现难点：**
- 主要难点在于正确地判断和处理第一个空闲页的情况，以及合理地使用`fetch_page_handle()`和`create_new_page_handle()`。


4. `void release_page_handle(RmPageHandle &page_handle);`

**方法声明：**
```c++
void release_page_handle(RmPageHandle &page_handle);
```

**方法实现思路：**
- 当`page`从已满变成未满的时候调用此函数。
- 提示：更新`page_hdr`的下一个空闲页和`file_hdr`的第一个空闲页。

**方法实现难点：**
- 主要难点在于正确地处理页面的状态变化，确保更新`page_hdr`和`file_hdr`的信息是正确的。


5. `RmFileHandle(DiskManager *disk_manager, BufferPoolManager *buffer_pool_manager, int fd);`

**方法声明：**
```c++
RmFileHandle(DiskManager *disk_manager, BufferPoolManager *buffer_pool_manager, int fd);
```

**方法实现思路：**
- 构造函数，当上层创建一个指向`RmFileHandle`的指针时将会在此初始化。
- 需要从磁盘中读出文件头的信息，即初始化`file_hdr`。
- 还需要设置该文件中开始分配的页面编号。

**方法实现难点：**
- 主要难点在于正确地初始化`file_hdr`，从磁盘中读取文件头的信息，以及合理地使用`DiskManager`和`BufferPoolManager`。


6. `std::unique_ptr<RmRecord> get_record(const Rid &rid, Context *context) const;`

**方法声明：**
```c++
std::unique_ptr<RmRecord> get_record(const Rid &rid, Context *context) const;
```

**方法实现思路：**
- 用于获取一条指定记录。由`Rid`得到`record`。
- 内部实现逻辑是把位于指定`slot`的`record`拷贝一份，然后返回给上层。

**方法实现难点：**
- 主要难点在于正确地获取指定记录，处理`Rid`和`slot`的映射关系，以及合理地使用`Context`。


7. `Rid insert_record(char *buf, Context *context);`

**方法声明：**
```c++
Rid insert_record(char *buf, Context *context);
```

**方法实现思路：**
- 用于插入一条指定记录。
- 对于堆文件组织形式，只需要找到一个有足够空间存放该记录的页面。
- 当所有已分配页面中都没有空间时，就申请一个新页面来存放该记录。
- 注意更新`bitmap`，它跟踪了每个`slot`是否存放了`record`；此外，如果当前`page`插入后已满，还需要更新`file_hdr`的第一个空闲页。

**方法实现难点：**
- 主要难点在于正确地插入记录，管理页面的状态和`bitmap`，以及合理地使用`Context`。


8. `void delete_record(const Rid &rid, Context *context);`

**方法声明：**
```c++
void delete_record(const Rid &rid, Context *context);
```

**方法实现思路：**
- 用于删除一条指定记录。使用`rid`中的`page_no`和`slot_no`。
- 先获取`page handle`，然后将`page`的`bitmap`中表示对应槽位的`bit`置0。
- 注意如果删除操作导致该页面恰好从已满变为未满，那么需要调用`release_page_handle()`。

**方法实现难点：**
- 主要难点在于正确地删除记录，处理`Rid`和`slot`的映射关系，以及合理地使用`Context`。


9. `void update_record(const Rid &rid, char *buf, Context *context);`

**方法声明：**
```c++
void update_record(const Rid &rid, char *buf, Context *context);
```

**方法实现思路：**
- 用于更新一条指定记录。
- 先获取`page handle`，然后直接更新`page`即

可。

**方法实现难点：**
- 主要难点在于正确地更新记录，处理`Rid`和`slot`的映射关系，以及合理地使用`Context`。

#### 实验发现

1. **`create_page_handle()` 方法**
   - **理论知识点**: 空闲页面的管理和分配。
   - **实现细节**: 判断并获取或创建一个空闲的`RmPageHandle`。
   - **理论对应**: 展示了页面管理中空闲页面的识别和利用，关联到数据库存储管理的基本概念。

2. **`release_page_handle(RmPageHandle &page_handle)` 方法**
   - **理论知识点**: 页面状态的更新和管理。
   - **实现细节**: 更新页面状态，包括`page_hdr`和`file_hdr`信息。
   - **理论对应**: 体现了页面状态管理的重要性，特别是在页面状态改变时如何更新相关信息。

3. **`RmFileHandle(DiskManager *disk_manager, BufferPoolManager *buffer_pool_manager, int fd)` 构造函数**
   - **理论知识点**: 文件句柄的初始化。
   - **实现细节**: 初始化`file_hdr`，设置文件中开始分配的页面编号。
   - **理论对应**: 关联到文件系统中文件句柄的初始化和文件头信息的管理。

4. **`get_record(const Rid &rid, Context *context) const` 方法**
   - **理论知识点**: 记录的检索。
   - **实现细节**: 根据`Rid`获取指定记录。
   - **理论对应**: 展示了如何在数据库系统中根据记录标识符定位并获取特定记录。

5. **`insert_record(char *buf, Context *context)` 方法**
   - **理论知识点**: 记录的插入和页面管理。
   - **实现细节**: 找到足够空间的页面或申请新页面来存放记录，更新`bitmap`和`file_hdr`。
   - **理论对应**: 体现了记录插入操作的复杂性，涵盖了页面空间管理和记录追踪的概念。



### 任务2.2: 记录迭代器

#### RmScan类的实现

1. `RmScan(const RmFileHandle *file_handle);`

**方法声明：**
```c++
RmScan(const RmFileHandle *file_handle);
```

**方法实现思路：**
- 传入`file_handle`，初始化`rid`，即设置初始迭代位置。
- 内部存放了`rid`，用于指向一个记录。

**方法实现难点：**
- 无特殊难点，主要是参数的正确传递和成员变量的初始化。


2. `void next() override;`

**方法声明：**
```c++
void next() override;
```

**方法实现思路：**
- 用于找到文件中下一个存放了记录的位置。
- 对于当前页面，可以使用`bitmap`来找到`bit`为1的`slot_no`。
- 如果当前页面的所有`slot`都没有存放记录，就找下一个页面。

**方法实现难点：**
- 主要难点在于正确找到下一个存放记录的位置，尤其是处理页面切换和`bitmap`的使用。


3. `bool is_end() const override;`

**方法声明：**
```c++
bool is_end() const override;
```

**方法实现思路：**
- 判断是否到达文件末尾，即最后一个页面的最后一个`slot`。
- 可以自主定义末尾的标识符，如`RM_NO_PAGE`。

**方法实现难点：**
- 主要难点在于正确判断是否到达文件末尾，需要考虑页面和`slot`的边界条件。


4. `Rid rid() const override;`

**方法声明：**
```c++
Rid rid() const override;
```

**方法实现思路：**
- 获取`RmScan`内部存放的`rid`，即当前迭代位置的记录。

**方法实现难点：**
- 无特殊难点，主要是返回内部存放的`rid`即可。

#### 实验发现

1. **`RmScan(const RmFileHandle *file_handle)` 构造函数**
   - **理论知识点**: 初始化迭代器，设置起始迭代位置。
   - **实现细节**: 传入`file_handle`并初始化`rid`以指向第一个记录。
   - **理论对应**: 展示了迭代器的初始化过程，符合数据库记录迭代的基本概念。

2. **`void next() override` 方法**
   - **理论知识点**: 寻找下一个有效记录的位置。
   - **实现细节**: 使用`bitmap`查找当前页面中的下一个有效`slot`，或跳转到下一个页面。
   - **理论对应**: 体现了数据库记录迭代过程中的页面切换和`bitmap`的使用，展示了如何高效地访问记录。

3. **`bool is_end() const override` 方法**
   - **理论知识点**: 判断迭代器是否已到达文件末尾。
   - **实现细节**: 检查是否到达最后一个页面的最后一个`slot`。
   - **理论对应**: 体现了如何在记录迭代器中处理边界条件，确保迭代过程的正确结束。

4. **`Rid rid() const override` 方法**
   - **理论知识点**: 获取当前迭代位置的记录标识符。
   - **实现细节**: 返回内部存放的`rid`。
   - **理论对应**: 展示了如何在记录迭代器中获取当前记录的位置信息。



## 实验2：索引管理器

### 任务1: B+树的查找

#### IxNodeHandle类的实现

1. `int lower_bound(const char *target) const`

**方法声明：**
```cpp
int lower_bound(const char *target) const;
```

**方法实现思路：**
- 这个方法的目的是在当前节点中找到第一个大于等于`target`的键的索引。
- 实现上采用了二分查找，以提高查找效率。
- 检查每个中间元素，比较它和`target`的大小，根据比较结果调整搜索范围。

**方法实现难点：**
- 实现二分查找时需要准确处理边界条件，确保不会漏掉任何可能的匹配。
- 确保在各种不同的节点配置（如不同的键数量）下都能正确工作。

2. `int upper_bound(const char *target) const`

**方法声明：**
```cpp
int upper_bound(const char *target) const;
```

**方法实现思路：**
- 类似于`lower_bound`，但这里寻找的是第一个大于`target`的键的索引。
- 依然使用二分查找来优化性能。
- 逻辑上稍有不同，更注重于找到比`target`大的第一个元素。

**方法实现难点：**
- 难点同样在于处理二分查找的边界条件，特别是在节点的键值接近`target`但略有差异的情况。

3. `bool LeafLookup(const char *key, Rid **value)`

**方法声明：**
```cpp
bool LeafLookup(const char *key, Rid **value);
```

**方法实现思路：**
- 在叶子节点中查找给定的`key`。
- 使用`lower_bound`定位可能的键位置。
- 验证找到的键是否确实匹配，如果匹配则提取相应的值。

**方法实现难点：**
- 关键在于正确处理不匹配的情况，如`key`不存在于节点中。

#### IxIndexHandle类的实现

1. `IxNodeHandle *FindLeafPage(const char *key, Operation operation, Transaction *transaction)`

**方法声明：**
```cpp
IxNodeHandle *FindLeafPage(const char *key, Operation operation, Transaction *transaction);
```

**方法实现思路：**
- 从根节点开始，逐层向下查找直到找到包含`key`的叶子节点。
- 在每个内部节点使用`InternalLookup`来确定下一层的子节点。

**方法实现难点：**
- 确保即使在大量节点和深层次的B+树中也能高效查找。

2. `bool GetValue(const char *key, std::vector<Rid> *result, Transaction *transaction)`

**方法声明：**
```cpp
bool GetValue(const char *key, std::vector<Rid> *result, Transaction *transaction);
```

**方法实现思路：**
- 使用`FindLeafPage`找到叶子节点。
- 调用`LeafLookup`来获取`key`对应的值。
- 处理查找结果，如存在则添加到结果向量中。

**方法实现难点：**
- 需要处理各种查找失败的情况，包括`key`不存在的情况。

#### 实验发现

IxNodeHandle类的实现
1. **`lower_bound(const char *target) const` 方法**
   - **理论知识点**: B+树中键的下界查找。
   - **实现细节**: 采用二分查找在节点中找到第一个大于等于目标的键的索引。
   - **理论对应**: 体现了B+树节点内部搜索的效率优化，特别是在有序键集中快速定位的能力。

2. **`upper_bound(const char *target) const` 方法**
   - **理论知识点**: B+树中键的上界查找。
   - **实现细节**: 类似于`lower_bound`但寻找第一个大于目标的键的索引。
   - **理论对应**: 展示了B+树在处理范围查询时的能力，如快速跳过小于目标值的所有键。

3. **`LeafLookup(const char *key, Rid **value)` 方法**
   - **理论知识点**: 在叶子节点中具体键值的查找。
   - **实现细节**: 在叶子节点使用`lower_bound`定位键的位置，并验证匹配后提取值。
   - **理论对应**: 体现了B+树叶子节点中进行精确查找的过程，展示了键到值映射的检索操作。

IxIndexHandle类的实现
1. **`FindLeafPage(const char *key, Operation operation, Transaction *transaction)` 方法**
   - **理论知识点**: B+树中的查找路径确定。
   - **实现细节**: 从根节点开始逐层向下查找，直至找到包含键的叶子节点。
   - **理论对应**: 展示了B+树多层结构中的逐层查找过程，从根节点到叶节点的路径确定。

2. **`GetValue(const char *key, std::vector<Rid> *result, Transaction *transaction)` 方法**
   - **理论知识点**: 键对应值的检索。
   - **实现细节**: 利用`FindLeafPage`找到叶子节点后，调用`LeafLookup`获取键对应的值。
   - **理论对应**: 体现了在确定了叶子节点后如何从中检索出具体的值，关联到B+树作为索引结构检索记录的实用性。


### 任务2: B+树的插入

#### IxNodeHandle类的实现

1. `void insert_pairs(int pos, const char *key, const Rid *rid, int n)`

**方法声明：**
```cpp
void insert_pairs(int pos, const char *key, const Rid *rid, int n);
```

**方法实现思路：**
- 在指定位置插入多个键值对。
- 方法首先检查插入位置的合法性，然后在数组中创建空间并插入新的键值对。
- 插入后更新节点的键数量。

**方法实现难点：**
- 在不违反B+树性质的情况下正确地移动现有的键值对，以插入新的键值对。

2. `int Insert(const char *key, const Rid &value)`

**方法声明：**
```cpp
int Insert(const char *key, const Rid &value);
```

**方法实现思路：**
- 如前文所述。

**方法实现难点：**
- 如前文所述。

#### IxIndexHandle类的实现

1. `void insert_entry(const char *key, const Rid &value)`

**方法声明：**
```cpp
void insert_entry(const char *key, const Rid &value);
```

**方法实现思路：**
- 找到适当的叶子节点进行插入。
- 处理可能的节点分裂。
- 在必要时更新父节点和根节点。

**方法实现难点：**
- 处理节点分裂和向上传播分裂产生的新键。

2. `IxNodeHandle *Split(IxNodeHandle *node)`

**方法声明：**
```cpp
IxNodeHandle *Split(IxNodeHandle *node);
```

**方法实现思路：**
- 分裂节点，将一个满节点拆分成两个节点。
- 更新指向新节点的指针和父节点中的引用。

**方法实现难点：**
- 保证分裂后的两个节点都符合B+树的性质，特别是关于节点大小的规则。

3. `void InsertIntoParent(IxNodeHandle *old_node, const char *key, IxNodeHandle *new_node, Transaction *transaction)`

**方法声明：**
```cpp
void InsertIntoParent(IxNodeHandle *old_node, const char *key, IxNodeHandle *new_node, Transaction *transaction);
```


**方法实现思路：**
- 在分裂节点后，将新节点的键插入到父节点。
- 如果父节点也满了，则递归地分裂父节点。

**方法实现难点：**
- 管理递归分裂的复杂性，确保在多层次分裂中保持树的平衡。

#### 实验发现


IxNodeHandle类的实现
1. **`insert_pairs(int pos, const char *key, const Rid *rid, int n)` 方法**
   - **理论知识点**: B+树节点中键值对的插入。
   - **实现细节**: 在指定位置插入多个键值对，并更新节点的键数量。
   - **理论对应**: 展示了在B+树节点中维护键值对序列的方法，特别是在插入新键值对时保持节点内部有序。

2. **`Insert(const char *key, const Rid &value)` 方法**
   - **理论知识点**: 单个键值对的插入操作。
   - **实现细节**: 实现逻辑与`insert_pairs`类似，专注于单个键值对的插入。
   - **理论对应**: 体现了在B+树节点中插入单个键值对的过程，保证了树结构的有序性和平衡。

IxIndexHandle类的实现
1. **`insert_entry(const char *key, const Rid &value)` 方法**
   - **理论知识点**: B+树的插入操作，包括找到适当节点和处理节点分裂。
   - **实现细节**: 找到合适的叶子节点进行插入，处理节点分裂并在必要时更新父节点和根节点。
   - **理论对应**: 展示了B+树插入过程中的关键步骤，如节点分裂和分裂后的父节点更新。

2. **`Split(IxNodeHandle *node)` 方法**
   - **理论知识点**: B+树的节点分裂操作。
   - **实现细节**: 实现逻辑未详细描述，但一般涉及将节点一分为二，平衡键值对分布。
   - **理论对应**: 关联到B+树在插入导致节点过满时如何维持树的结构和平衡。

3. **`InsertIntoParent(IxNodeHandle *old_node, const char *key, IxNodeHandle *new_node, Transaction *transaction)` 方法**
   - **理论知识点**: 分裂后的节点插入到父节点。
   - **实现细节**: 将分裂产生的新节点的键插入到父节点，如果父节点满了，则递归地分裂父节点。
   - **理论对应**: 展示了B+树在节点分裂后如何更新其父节点，保持整个树的平衡和有序性。


### 任务3: B+树的删除

#### IxNodeHandle类的实现

1. `void erase_pair(int pos)`

**方法声明：**
```cpp
void erase_pair(int pos);
```

**方法实现思路：**
- 在指定位置删除键值对。
- 更新节点的键数量。

**方法实现难点：**
- 在删除键值对后保持节点键的连续性。

2. `int Remove(const char *key)`

**方法声明：**
```cpp
int Remove(const char *key);
```

**方法实现思路：**
- 找到键所在位置并删除。
- 如前文所述。

**方法实现难点：**
- 如前文所述。

#### IxIndexHandle类的实现

1. `void delete_entry(const char *key, Transaction *transaction)`

**方法声明：**
```cpp
void delete_entry(const char *key, Transaction *transaction);
```

**方法实现思路：**
- 找到并删除指定键。
- 在必要时进行节点合并或重新分配。

**方法实现难点：**
- 确保删除后的节点仍然符合B+树的性质。

2. `bool CoalesceOrRedistribute(IxNodeHandle *node, Transaction *transaction)`

**方法声明：**
```cpp
bool CoalesceOrRedistribute(IxNodeHandle *node, Transaction *transaction);
```

**方法实现思路：**
- 评估删除后是否需要合并或重新分配。
- 执行相应的操作以保持树的平衡。

**方法实现难点：**
- 评估并决定是合并还是重新分配，确保操作后树的结构正确。

3. `bool Coalesce(IxNodeHandle **neighbor_node, IxNodeHandle **node, IxNodeHandle **parent, int index, Transaction *transaction)`

**方法声明：**
```cpp
bool Coalesce(IxNodeHandle **neighbor_node, IxNodeHandle **node, IxNodeHandle **parent, int index, Transaction *transaction);
```

**方法实现思路：**
- 合并两个节点，并更新父节点的键和指针。
- 在必要时递归向上进行合并。

**方法实现难点：**
- 处理多层次的合并和父节点的更新。

4. `void Redistribute(IxNodeHandle *neighbor_node, IxNodeHandle *node, IxNodeHandle *parent, int index)`

**方法声明：**
```cpp
void Redistribute(IxNodeHandle *neighbor_node, IxNodeHandle *node, IxNodeHandle *parent, int index);
```

**方法实现思路：**
- 在两个兄弟节点间重新分配键值对，以避免节点合并。
- 更新父节点的键和指针。

**方法实现难点：**
- 确保重新分配后节点大小符合B+树性质。

5. `bool AdjustRoot(IxNodeHandle *old_root_node)`

**方法声明：**
```cpp
bool AdjustRoot(IxNodeHandle *old_root_node);
```

**方法实现思路：**
- 调整根节点，在删除操作后可能需要降低树的高度。
- 如果根节点变为空，则设置其唯一的子节点为新的根节点。

**方法实现难点：**
- 在调整根节点时保持树的整体结构和性质。

#### 实验发现

IxNodeHandle类的实现
1. **`erase_pair(int pos)` 方法**
   - **理论知识点**: B+树节点中键值对的删除。
   - **实现细节**: 在指定位置删除键值对，然后更新节点的键数量。
   - **理论对应**: 展示了在B+树节点中维护键值对序列的方法，特别是在删除键值对时保持节点内部连续性。

2. **`Remove(const char *key)` 方法**
   - **理论知识点**: 根据键删除节点中的键值对。
   - **实现细节**: 定位到键所在位置并进行删除操作。
   - **理论对应**: 体现了在B+树节点中执行特定键的精确删除操作。

IxIndexHandle类的实现【68†source】
1. **`delete_entry(const char *key, Transaction *transaction)` 方法**
   - **理论知识点**: 从B+树中删除特定键。
   - **实现细节**: 实现逻辑未详细描述，但一般涉及定位到键所在的叶节点，并执行删除操作。
   - **理论对应**: 关联到B+树中的键删除流程，包括查找、删除及其后续操作。

2. **`CoalesceOrRedistribute(IxNodeHandle *node, Transaction *transaction)` 方法**
   - **理论知识点**: 删除后的节点合并或重新分配。
   - **实现细节**: 根据节点删除后的状态，决定是合并还是重新分配节点，以保持树的平衡。
   - **理论对应**: 展示了B+树在删除操作后如何处理节点以维持树结构的平衡性。

3. **`Coalesce(IxNodeHandle **neighbor_node, IxNodeHandle **node, IxNodeHandle **parent, int index, Transaction *transaction)` 方法**
   - **理论知识点**: 节点的合并操作。
   - **实现细节**: 实现逻辑未详述，但一般涉及将两个兄弟节点合并为一个节点，以及对父节点的相应更新。
   - **理论对应**: 展示了B+树在节点过小或键值过少时，如何通过合并节点来保持树的结构和效率。

