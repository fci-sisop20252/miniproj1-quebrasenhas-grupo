# Relatório: Mini-Projeto 1 - Quebra-Senhas Paralelo

**Aluno(s):** Nome (Matrícula), Nome (Matrícula),,,  
---

## 1. Estratégia de Paralelização


**Como você dividiu o espaço de busca entre os workers?**

[Explique seu algoritmo de divisão]

**Código relevante:** Cole aqui a parte do coordinator.c onde você calcula a divisão:
```c
long long passwords_per_worker = total_space / num_workers;
    long long remaining = total_space % num_workers;
```

---

## 2. Implementação das System Calls

**Descreva como você usou fork(), execl() e wait() no coordinator:**

duplicamos o processo com o fork() e salvamos o pid para o tratamento dos processos, utilizamos o pid para identificar o processo filho e o pai. Os processos filhos são tratados com o execl(), que passa a executar o worker do arquivo worker. O processo pai armazena o pid dos processos filhos em uma lista. o wait() faz com que o processo pai aguarde o retorno do processo filho, e utilizamos a variavel de status para guardar e validar o retorno do filho.


**Código do fork/exec:**
```c
for (int i = 0; i < num_workers; i++) {
        long long chunk_size = passwords_per_worker + (i < remaining ? 1 : 0);
        long long end_index = start_index + chunk_size - 1;
        // TODO: Calcular intervalo de senhas para este worker
        // TODO: Converter indices para senhas de inicio e fim
        char start_password[64];
        char end_password[64];
        index_to_password(start_index, charset, charset_len, password_len, start_password);
        index_to_password(end_index, charset, charset_len, password_len, end_password);
        // TODO 4: Usar fork() para criar processo filho
        // TODO 5: No processo pai: armazenar PID
        // TODO 6: No processo filho: usar execl() para executar worker
        // TODO 7: Tratar erros de fork() e execl()
        pid_t pid = fork();
        if (pid < 0) {
            perror("Erro em fork");
            exit(1);
        }

        if (pid == 0) {
            execl("./worker", "worker",
                  target_hash,
                  start_password,
                  end_password,
                  charset,
                  argv[2], 
                  (char[16]){0}, 
                  NULL);

            perror("Erro em execl");
            exit(1);
        } 
        else {
            workers[i] = pid;
        }
        start_index = end_index + 1;
    }
```

---

## 3. Comunicação Entre Processos

**Como você garantiu que apenas um worker escrevesse o resultado?**

a funcao save_result trata esse problema atraves do open()

**Como o coordinator consegue ler o resultado?**

Através do arquivo password_found.txt, utilizando o comando fopen() e os comandos de leitura fgets() e sscanf()
---

## 4. Análise de Performance
Complete a tabela com tempos reais de execução:
O speedup é o tempo do teste com 1 worker dividido pelo tempo com 4 workers.

| Teste | 1 Worker | 2 Workers | 4 Workers | Speedup (4w) |
|-------|----------|-----------|-----------|--------------|
| Hash: 202cb962ac59075b964b07152d234b70<br>Charset: "0123456789"<br>Tamanho: 3<br>Senha: "123" | 0.006s | 0.008s | 0.006s | 1 |
| Hash: 5d41402abc4b2a76b9719d911017c592<br>Charset: "abcdefghijklmnopqrstuvwxyz"<br>Tamanho: 5<br>Senha: "hello" | 5s | 7s | 1s | 5 |

**O speedup foi linear? Por quê?**
não, fica visivel no segundo teste que 2 workers demoraram mais do que apenas 1 para encontrar a senha. isso acontece por causa da divisao do espaço de busca e da localizacao da senha nesse espaço.

---

## 5. Desafios e Aprendizados
**Qual foi o maior desafio técnico que você enfrentou?**

o meu maior desafio foi o loop para a criação dos workers e a divisao do trabalho.
---

## Comandos de Teste Utilizados

```bash
# Teste básico
./coordinator "900150983cd24fb0d6963f7d28e17f72" 3 "abc" 2

# Teste de performance
time ./coordinator "202cb962ac59075b964b07152d234b70" 3 "0123456789" 1
time ./coordinator "202cb962ac59075b964b07152d234b70" 3 "0123456789" 4

# Teste com senha maior
time ./coordinator "5d41402abc4b2a76b9719d911017c592" 5 "abcdefghijklmnopqrstuvwxyz" 4
```
---

**Checklist de Entrega:**
- [ ] Código compila sem erros
- [ ] Todos os TODOs foram implementados
- [ ] Testes passam no `./tests/simple_test.sh`
- [ ] Relatório preenchido
