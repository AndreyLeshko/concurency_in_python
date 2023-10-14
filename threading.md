# threading

```commandline
    import threading
    
    def thread_func(thread_num):
        print(f'Hello from thread {thread_num}')
    
    if __name__ == '__main__':
        t1 = threading.Thread(target=thread_func, args=(1,), daemon=False)
        t2 = threading.Thread(target=thread_func, args=(2,), daemon=False)
        t1.start()
        t2.start()
```

Что мы здесь видим?. 

Мы можем выполнить функцию в отдельном потоке. Создаём объект Thread, передаем саму функцию (функцию! Не надо здесь её вызывать, иначе функция сразу выполнится в текущем потоке, а передадите вы результат её выполнения).
Далее передаём аргументы в кортеже (даже если аргумент всего один, передать его нужно в кортеже).
А что за аргумент daemon? Об этом чуть ниже.

Мы создали поток... но не совсем. Мы создали объект Thread, а это лишь внутренний объект python.
Операционная система о новом потоке ещё ничего не знает. Далее мы вызываем у объекта метод start, поток создается и начинает исполняться.


Теперь о демонах... Аргумент daemon=False указывает, что дочерний поток не выступает демон (фоновым) процессом.
Если вкратце, у нас есть главный поток и 2 дочерних. Если задачи главного потока выполнятся быстрее, чем завершатся дочерние потоки, программа будет ждать завершение дочерних not-daemon потоков.
Если какой-то из потоков запущен как демон, завершение главного потока приведет к принудительному завершению демон потока.
Чтобы понять отличие между ними, посмотрите результат двух нижеприведенных скриптов.

```commandline
import threading
import time


def not_daemon():
    for i in range(5):
        print(f'Not daemon thread {i}')
        time.sleep(1)
    print('Finish not-daemon thread')

t = threading.Thread(target=not_daemon, args=(), daemon=False)
t.start()

time.sleep(3)

print('Finish main thread')


# Not daemon thread 0
# Not daemon thread 1
# Not daemon thread 2
# Finish main thread
# Not daemon thread 3
# Not daemon thread 4
# Finish not-daemon thread
```

```commandline
import threading
import time

def daemon():
    for i in range(5):
        print(f'Daemon thread {i}')
        time.sleep(1)
    print('Finish daemon thread')

t = threading.Thread(target=daemon, args=(), daemon=True)
t.start()

time.sleep(3)

print('Finish main thread')


# Not daemon thread 0
# Not daemon thread 1
# Not daemon thread 2
# Finish main thread
```

В первом варианте главный поток завершил работу, дождался выполнения дочернего потока и затем программа завершилась.
Во втором варианте, после завершения выполнения главного потока завершается вся программа, дочерний поток завершается принудительно.

Дождаться выполнения not-daemon потока можно вызвав у него метод t.join(). 
По сути, not-daemon поток эквивалентен daemon потоку + join(), однако основное назначение join-а в другом, 
он указывает конкретное место в программе где мы должны дождаться завершения дочернего потока. 
Например, мы можем запустить параллельно несколько потоков с разными задачами, затем вызвав join у каждого потока, дождаться их завершения, получить результаты и проделать с ними какую-то работу уже в главном потоке.

Демон потоки следует использовать для фоновых, некритичных задач. Они не хранят важные данные и должны быть устойчивы к произвольному завершению работы.

