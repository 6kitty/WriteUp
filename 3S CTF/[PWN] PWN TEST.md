prob.py 

import os
import random
import subprocess
import base64
import string
import uuid

def compile(size):
    code = f'''
#include<stdio.h>
#include <unistd.h>
#include <signal.h>
#include <stdlib.h>

void win()
{{
    system("/bin/sh");
}}

void handle_timeout(int sig) 
{{
    puts("Time out!");
    exit(1);
}}

int main()
{{
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);
    signal(SIGALRM, handle_timeout);
    alarm(10);
    
    char a[{size}];
        
    gets(a);
    puts(a);
    gets(a);

    return 0;
}}
'''
    return code

def generate_random_string(length):
    letters = string.ascii_letters + string.digits
    result_str = ''.join(random.choice(letters) for i in range(length))
    return 'flag{' + result_str + '}'

def main():
    n = random.randint(10, 20)
    print("Welcome to easy pwn test")
    print(f"You have to pass {n} tests")
    ok = input("If you are ready, please enter 3S\n-> ")
    assert ok == "3S"
    for i in range(n):
        print(f"test {i + 1}")
        unique_id = str(uuid.uuid4())
        source_file = f'{unique_id}.c'
        executable = f'{unique_id}'
        try:
            with open(source_file, 'w') as cfile:
                cfile.write(compile(random.randint(0x10, 0x50)).strip())
            compile_process = subprocess.run(['gcc', '-no-pie', '-z relro', source_file, '-o', executable], 
                                         stdout=subprocess.PIPE, stderr=subprocess.PIPE, timeout=5)
            assert compile_process.returncode == 0
            os.system(f'strip {executable}')
            with open(executable, 'rb') as prob:
                print(base64.b64encode(prob.read()))
            flag_str = generate_random_string(random.randint(0x18, 0x28))
            with open(f'flag', 'w') as flag:
                flag.write(flag_str)
            os.system(f'./{executable}')
            check = input("flag -> ")
            assert check == flag_str
        except AssertionError:
            print(f"test {i + 1} fail")
            exit()
        finally:
            os.system(f'rm flag {source_file} {executable}')
    print("3S{Your_Python_skill_is_good!}")

if __name__ == "__main__":
    main()

코드를 뜯어보자 
뭐지 아래서부터 보는데 플래그가 그대로 있긴 하다 

ok = input("If you are ready, please enter 3S\n-> ")
assert ok == "3S"

일단 여기서 3S를 입력하면 

            with open(source_file, 'w') as cfile:
                cfile.write(compile(random.randint(0x10, 0x50)).strip())
            compile_process = subprocess.run(['gcc', '-no-pie', '-z relro', source_file, '-o', executable], 
                                         stdout=subprocess.PIPE, stderr=subprocess.PIPE, timeout=5)
            assert compile_process.returncode == 0
            os.system(f'strip {executable}')

compile 함수에 있는 내용 컴파일 

            with open(executable, 'rb') as prob:
                print(base64.b64encode(prob.read()))
            flag_str = generate_random_string(random.randint(0x18, 0x28))
            with open(f'flag', 'w') as flag:
                flag.write(flag_str)
            os.system(f'./{executable}')
            check = input("flag -> ")
            assert check == flag_str

코드가 복잡해보이지만 풀어보면 결국 check와 flag_str은 같을 수밖에 없다 
그리고 그냥 rm된다 음?

 print("3S{Your_Python_skill_is_good!}")

 3S 포맷이 저 플래그밖에 없다  
