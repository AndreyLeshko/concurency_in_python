# multiprocessing

Переходим к мультипроцессности (Не мультипроцессорность! Не путайте понятия).

Для прочтения данного раздела, следует ознакомиться с разделом **threading**.
Конструкции запуска процессов и потоков в python очень схожи, поэтому не буду повторяться, а уделю больше внимания различиям с потоками.

Разберем также простенький пример:

```commandline
    from multiprocessing import Process
    
    def process_func(process_num):
        print(f'Hello from process {process_num}')
    
    if __name__ == '__main__':
        p1 = Process(target=process_func, args=(1,), daemon=False)
        p2 = Process(target=process_func, args=(2,), daemon=False)
        p1.start()
        p2.start()
```



