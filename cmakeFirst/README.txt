This directory contains source code examples for the CMake Tutorial.
Each step has its own subdirectory containing code that may be used as a
starting point. The tutorial examples are progressive so that each step
provides the complete solution for the previous step.

1. 安装cmake 
ubuntu : sudo apt install cmake 
windows 官网下载x64版本安装包


2. Basic 
  p1.设置cmake版本  cmake_minimum_required(VERSION ***) 
  p2.设置项目名（也就是可执行文件名）  project(*** VERSION 1.0)
  p3.设置可执行文件有什么源文件编译链接而来 add_executable(*** .cpp .cpp) 


3. 指定cpp标准
  p1.设置cpp版本 set(CMAKE_CXX_STANDARD 14)
  p2.要求编译器必须使用要求的cpp版本 set(CMAKE_CXX_STANDARD_REQUIRED True) 


4. cpp与CMakeLists.txt传递消息 
  p1.message(STATUS "hello world from cmake") 构建过程中输出消息到终端
  p2.configure_file(*.h.in *.h )
  p3.target_include_directories(** PUBLIC )指定可执行文件需要include头文件路径


5. 生成自己的库，链接自己的库
  p1. add_library(库名 .cpp .cpp) 在库cpp文件目录下的CMakeLists.txt 
  p2. target_link_libraries(Tutorial PUBLIC MathFunctions) 链接库
  p3. target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           "${PROJECT_SOURCE_DIR}/MathFunctions"
                           ) 指定库的文件所在目录
  p4. 在cpp文件中include头文件，并调用库函数

option(USE_MYMATH "Use my math?" OFF)  判断USE_MYMATH来进行选择编译


6. cmakedefine和define
  对于布尔值false 常量0 未定义变量 的话，cmakedefine视为无定义


8. 链接库视为可选 
  if(USE_MYMATH)
    *** 
  endif 


9. 为库添加使用依赖,链接库时不需要Include头文件目录
  target_include_directories(MathFunctions PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

PUBLIC 本目标要有，依赖本目标的要有
INTERFACE 本目标可以不用，依赖本目标的要有
PRIVATE 本目标要有，依赖本目标的不可以访问 


10. 接口库设置cpp标准
  add_library(** INTERFACE .cpp) 
  target_compile_features(** INTERFACE cxx_std_14) 
  target_link_libraries(TUtorial PUBLIC **) 


11. 生成器表达式 generator expressions 
  cmake配置阶段 cmake -G "MinGW Makefils" .. 
  cmake生成阶段 cmake --build . 

    $<condition:true_string> condition为1则返回true_string，否则无返回
    

12. 安装规则 
  install(TARGETS 被安装 DESTINATION 目标目录)  安装可执行文件
  命令行运行cmake --install .  安装到 "当前目录/目标目录" 
          cmake --install . prefix=path 安装到path/目标目录 

  set(installable_libs 库) 
  install(TARGETS ${installable_libs} DESTINATION libs) 安装库

  install(FILES ${PROJECT_BINARY_DIR}/.h DESTINATION include) 安装头文件 


13. 测试 
  p1. enable_testing() 开启测试 
  p2. add_test(NAME Runs COMMAND Tutorial 25) 
  p3. add test(Name Usage COMMAND Tutorial)
      set_tests_property(Usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage.*number") 

      add_test(NAME Com4 COMMAND Tutorial 4) 
      set_tests_property(Com4 PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2") 

  p4.对多个输入 1 4 9 16 25 36 等测试
    function(do_test num result)
      add_test(NAME COM${num} COMMAND Tutorial ${num}) 
      set_tests_property(COM${num} PROPERTIES PASS_REGULAR_EXPRESSION "${num} is ${result}") 
    endfunction 

    do_test(1 1)
    do_test(4 2)
    do_test(9 3)
    do_test(16 4)
    do_test(25 5)
    do_test(-25 "(-nan|nan|0)")
    do_test(0.01 0.1)
    命令行运行ctest -VV 得到多测试用例结果 
    

14. 系统特性检测 
  p1. include(CheckCXXSourceCompiles) 
  p2. check_cxx_source_compile("
      c++源代码
      #include<iostream> 
      int main()
      {
        std::log(1.0);
        return 0;
      }
  "  HAVE_LOG)
  p3. if (HAVE_LOG AND HAVE_EXP)
        target_compile_definitions(目标库MathFunctions PRIVATE "HAVE_LOG" "HAVE_EXP")
      endif 

      生成库的源代码中插入：
      #if defined(HAVE_EXP) && defined(HAVE_LOG) 
          std::log(2.5) + std::exp(2) ******** 
      #endif 


15. 添加自定义命令 
  g++ makeTable.cpp -o makeTable 将源文件编译为可执行文件 
  add_executable(makeTable makeTable.cpp)
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h 
    COMMAND makeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h 
    DEPENDS makeTable 
  )


16. 打包程序 
  NSIS格式适应于windows，配合cmake打包为exe文件 
# 添加可执行目标
add_executable(MyTarget main.cpp)

# 复制资源文件到构建目录
file(COPY resources DESTINATION ${CMAKE_BINARY_DIR})

# 安装目标文件和资源文件
install(TARGETS MyTarget DESTINATION bin)
install(DIRECTORY resources DESTINATION bin)

# 设置CPack配置
set(CPACK_PACKAGE_NAME "MyProject")                  # 设置打包的项目名称
set(CPACK_PACKAGE_VERSION "1.0.0")                  # 设置打包的项目版本
set(CPACK_PACKAGE_VENDOR "MyVendor")                 # 设置打包的项目厂商信息
set(CPACK_PACKAGE_DESCRIPTION "My Project Description")    # 设置打包的项目描述信息
set(CPACK_PACKAGE_CONTACT "My Contact Info")         # 设置打包的联系方式
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}")  # 设置打包文件名
set(CPACK_GENERATOR ZIP)                             # 设置生成ZIP格式的安装包
set(CPACK_SOURCE_GENERATOR ZIP)                      # 设置生成ZIP格式的源码包

# 调用CPack命令
include(CPack)

在build文件夹命令行输入cpack -G ZIP(or 其他格式) 命令打包



17.动态库 静态库 
  首先在生成库时候将add_library(SqrtLibrary SHARED .cpp) 其中SHARED为动态库
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}) 指定可执行文件输出目录
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}) 指定.a .lib输出目录
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}) linux平台输出.so目录


18. 链接第三方库 
  p1.指定库位置 set(MATH_DIR path) 
  p2.指定头文件路径 target_include_directories(Tutorial PRIVATE ${MATH_DIR}/include) 
  p3.指定链接库位置 target_link_directories(Tutorial PRIVATE ${MATH_DIR}/lib)
  p4.指定需要链接的库 target_link_libraries(Tutorial PRIVATE MathFunctions) 


19.链接使用cmake管理的第三方库(如opencv) 
  p1.set(Opencv_DIR path) 
  p2.find_package(Opencv REQUIRED) 
  p3.target_link_libraries(demo ${Opencv_LIBS}) 
  

20. 打包调试和发行版 
  p1. mkdir debug 
      cd debug 
      cmake -DCMAKE_BUILD_TYPE=Debug .. 
  p2. mkdir release 
      cd release 
      cmake -DCMAKE_BUILD_TYPE=Release .. 


21. cmake-gui 


22. cmake命令行
  cmake -S src -B build 不在代码目录下时 
  cmake --build . -j4 多线程构建 
  cmake --build . -t clean 清楚上次构建再构建 
  cmake -P **.cmake  运行cmake脚本


