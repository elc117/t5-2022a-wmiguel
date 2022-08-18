# Estudo de Multithreading e Multiprocessamento em Python

## Motivação:
Curiosidade sobre a utilização de threads despertada após realização dos exercícios propostos pela professora, objetivando aprender mais sobre 
seu uso, vantagens, desvantagens, e melhor distinção entre as diferenças de ambos.

## Por que usar Threads?
Threads são uma forma de fazer com que a nossa aplicação execute tarefas de forma assíncrona, por exemplo, enquanto uma estrutura de repetição 
é executada podemos executar uma outra rotina.

## Multithreading e Multiprocessamento
Multithreading e multiprocessamento são duas maneiras de alcançar multitarefa que é muito útil na execução de funções e código em
paralelo.

### Multithreading X Multiprocessamento

Multithreading refere-se à capacidade de um processador executar vários threads simultaneamente, onde cada thread executa um processo. Enquanto
o multiprocessamento se refere à capacidade de um sistema executar vários processadores simultaneamente, onde cada processador pode executar um 
ou mais threads.

O multithreading é útil para processos vinculados a IO, como a leitura de arquivos de uma rede ou banco de dados, pois cada thread pode executar 
o processo vinculado a IO simultaneamente. O multiprocessamento é útil para processos vinculados à CPU, como tarefas computacionalmente pesadas, 
pois se beneficiará de ter vários processadores, semelhante a como os computadores multicore funcionam mais rápido do que os computadores com um 
único núcleo.

### Multithreading como uma função em Python

O multithreading pode ser implementado usando a biblioteca interna do Python threading e possui a seguinte ordem:

* Criar encadeamento
* Iniciar a execução da tarefa
* Aguardar a execução do thread

#### Implementação:

Obs: Comentários foram feitos no código, a fim de explicar um pouco sobre as descobertas e pontuações de coisas já compreendidas.

```
import os
import threading
import time


def task_sleep(sleep_duration, task_number, lock):
    lock.acquire()
    # Se time.sleep fosse implementado aqui, as threads seriam executadas 
    # sequencialmente e não haveria economia de tempo, para realizarmos esse 
    # teste, é só mover nosso time.sleep para cá >.<
    lock.release()

    time.sleep(sleep_duration)
    # Processos executados em threads diferentes mas com o mesmo processador 
    print(f"Tarefa {task_number} completa. "
          f"Main thread: {threading.main_thread().name}, "
          f"Thread atual: {threading.current_thread().name}, "
          f"ID do processo: {os.getpid()}\n")


if __name__ == "__main__":
    time_start = time.time()

    # Cria lock (opcional)
    thread_lock = threading.Lock()

    # Cria thread
    t1 = threading.Thread(target=task_sleep, args=(1, 1, thread_lock))
    t2 = threading.Thread(target=task_sleep, args=(1, 2, thread_lock))
    t3 = threading.Thread(target=task_sleep, args=(1, 3, thread_lock))

    # Comeca a execucao
    t1.start()
    t2.start()
    t3.start()

    # Aguarda a thread para completar a execucao
    t1.join()
    t2.join()
    t3.join()

    time_end = time.time()
    print(f"Time elapsed: {round(time_end - time_start, 2)}s")

```

### Multithreading como uma classe em Python

o multithreading pode ser implementado como uma classe Python que herda da threading.Threadsuperclass. Um benefício de usar classes em vez de funções 
seria a capacidade de compartilhar variáveis por meio de objetos de classe.

A diferença entre implementar multithreading como uma função X classe estaria na etapa 1 (Criar Thread), já que thread agora está marcada com um método 
de classe em vez de uma função. As etapas subsequentes t1.start() e t1.join() permanecem as mesmas.

#### Implementação:

```

import threading
import time

class Sleep(threading.Thread):
    def __init__(self, sleep_duration):
        self.sleep_duration = sleep_duration

    def sleep(self):
        time.sleep(self.sleep_duration)

if __name__ == "__main__":
    # Cria thread
    sleep_class = Sleep(2)
    t1 = threading.thread(target=sleep_class.sleep)
    
```

### Multiprocessamento como uma função em Python

O multiprocessamento pode ser implementado com a biblioteca interna do Python multiprocessingusando dois métodos diferentes, Process e Pool.

O método Process é semelhante ao método multithreading. Ao rodarmos o código, podemos ver que o tempo gasto é maior para multiprocessamento 
do que multithreading, pois há mais sobrecarga na execução de vários processadores.

#### Implementação:

```

import multiprocessing
import os
import time


def task_sleep(sleep_duration, task_number):
    # Processos executados em diferentes processadores, como podemos ver no print em "ID do precesso" 
    time.sleep(sleep_duration)
    print(f"Task {task_number} done (slept for {sleep_duration}s)! "
          f"Process ID: {os.getpid()}\n")


if __name__ == "__main__":
    time_start = time.time()

    # Cria process
    p1 = multiprocessing.Process(target=task_sleep, args=(1, 1))
    p2 = multiprocessing.Process(target=task_sleep, args=(1, 2))
    p3 = multiprocessing.Process(target=task_sleep, args=(1, 3))

    # Comeca execucao
    p1.start()
    p2.start()
    p3.start()

    # Aguarda o process para completar a execucao
    p1.join()
    p2.join()
    p3.join()

    time_end = time.time()
    print(f"Time elapsed: {round(time_end - time_start, 2)}s")

```

O método de pool permite que a gente defina o número de trabalhadores e distribua todos os processos para os processadores, manipulando o agendamento
de processos automaticamente. O método Pool é usado para dividir uma função em várias partes pequenas, executando a mesmafunção com diferentes argumentos 
de entrada.

#### Implementação:

```

import multiprocessing
import os
import time


def task_sleep(sleep_duration, task_number):
    time.sleep(sleep_duration)
    print(f"Task {task_number} done (slept for {sleep_duration}s)! "
          f"Process ID: {os.getpid()}")


if __name__ == "__main__":
    time_start = time.time()

    # Criar pool de trabalhadores
    pool = multiprocessing.Pool(2)

    # O método Pool é usado para dividir uma função em várias partes pequenas usando starmap nesse caso, mas eu vi que pode ser usado o map também
    pool.starmap(func=task_sleep, iterable=[(2, 1)] * 10) 
    

    # Aguarde até que os trabalhadores concluam a execução
    pool.close()

    time_end = time.time()
    print(f"Time elapsed: {round(time_end - time_start, 2)}s")
    
```
Considerações: Podemos perceber que  o tempo gasto é maior para os nossos códigos utilizando multiprocessamento em relação ao código utilizando multithreads, 
pois a sobrecarga para gerenciar vários processos é maior do que para gerenciar vários threads, devido ao uso de vários núcleos de CPU pelo programa.


Conclusão: 

Notamos que a as vantagens na utilização de threads é a facilidade no desemvolvimento, visto que torna possível elaborar e criar o programa em módulos, 
experimentando-os isoladamente no lugar de escrever em um único bloco de código. Outro benefício dos threads é que eles não deixam o processo parado, pois 
quando um deles está aguardando um determinado dispositivo de entrada ou saída, ou ainda outro recurso do sistema, outro thread pode estar trabalhando. 
No entanto, uma das desvantagens é que com vários threads o trabalho fica mais complexo, justamente por causa da interação que ocorre entre eles. 

## Referências

https://www.geeksforgeeks.org/multithreading-python-set-1/
https://www.geeksforgeeks.org/difference-between-multithreading-vs-multiprocessing-in-python/?ref=rp

Os códigos utlizados pertencem a esse repositório, sofreram adição de comentários sobre a execução do programa e pequenas alterações.
https://gist.github.com/kayjan
