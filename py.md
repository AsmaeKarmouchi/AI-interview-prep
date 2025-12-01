try:
    # code that may raise an exception
except SomeException as e:
    # code to handle the exception


import decimal
integer = 10
print(decimal.Decimal(integer))
string = '12345'
print(decimal.Decimal(string))

### Reversing a String using an Extended Slicing Technique
string = "Python Programming"
print(string[::-1])

lst = ["P", "Y", "T", "H", "O", "N"]
string = ''.join(lst)
print(string)
### Counting the White Spaces in a String
string = "P r ogramm in g "
print(string.count(' '))

### The with statement simplifies exception handling by encapsulating common preparation and cleanup tasks in so-called context managers.
with open('file.txt', 'r') as file:
    data = file.read()

##
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instances[cls]

class MyClass(metaclass=Singleton):
    pass

##
import threading

def print_numbers():
    for i in range(10):
        print(i)

thread = threading.Thread(target=print_numbers)
thread.start()
thread.join()


## Les coroutines sont un type de fonction permettant de suspendre et de reprendre leur exécution. Elles sont définies async defet utilisées awaitpour redonner le contrôle à la boucle d'événements.
import asyncio

async def say_hello():
    await asyncio.sleep(1)
    print("Hello")

asyncio.run(say_hello())

# 
def is_valid(s):
    stack = []
    mapping = {')': '(', '}': '{', ']': '['}
    for char in s:
        if char in mapping.values():
            stack.append(char)
        elif char in mapping:
            if not stack or stack[-1] != mapping[char]:
                return False
            stack.pop()
        else:
            return False
    return not stack

