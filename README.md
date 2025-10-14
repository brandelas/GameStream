# GameStream
# Plano de Arquitetura Inicial para o GameStream

**Arquiteto de Soluções:** Eloísa dos Santos Brandão
**RA:** 2325096

## Nosso Objetivo

Queremos construir o GameStream para aguentar 1 milhão de usuários e **NUNCA CAIR**. Para isso, vamos dividir o sistema em partes menores e garantir que ele funcione em vários lugares ao mesmo tempo.

---

## 1. Dividindo o Trabalho (Microsserviços)

A ideia de "Microsserviços" é simples: se uma parte falhar, ela não pode levar o sistema inteiro junto. Separamos o que é menos importante do serviço principal de streaming.

### Microsserviços Separados que Não Podem Falhar o Jogo:

| Microsserviço | Para que Serve | Por que Isolamos? |
|
| **1. Login e Senha (Autenticação)** | Controla quem entra no serviço. | Se falhar, quem já está jogando continua jogando! Apenas novos usuários não conseguem entrar por um tempo. |
| **2. Catálogo de Jogos** | Mostra todos os jogos disponíveis para você escolher. | Se essa lista sumir temporariamente, o jogo de quem já começou não para. É um ponto de falha que não afeta o serviço principal. |

### Comunicação Segura entre as Partes:
Para garantir que a falha de um não afete o outro, eles não vão conversar diretamente.
* **O que usar:** **Fila de Mensagens Gerenciada (Tipo SQS/Pub/Sub)**.
* **Como funciona:** Se o Serviço A precisar mandar algo para o Serviço B, ele coloca a mensagem numa "fila de espera". Se o Serviço B estiver ocupado ou fora do ar, a fila segura a mensagem até que ele volte. **É como deixar um recado que não se perde.**

---

## 2. Garantindo que Nunca Pare (Resiliência)

Para que o GameStream fique sempre no ar (o que chamamos de "Resiliência" e "Disponibilidade"):

### Onde Colocar os Servidores?
* **O que usar:** **Múltiplas Regiões e Múltiplas Zonas de Disponibilidade (Multi-AZ)**.

### Por que fazer isso?
1. **Proteção Local (Multi-AZ):** Se um centro de dados (datacenter) na sua cidade sofrer uma queda de energia ou incêndio, o outro centro de dados vizinho assume na hora.
2. **Proteção Global (Multi-Regiões):** Se uma região inteira (o Brasil, por exemplo) tiver um problema gigante, o serviço muda automaticamente para a região vizinha (EUA ou Europa). Isso também **reduz o atraso (latência)**, que é crucial para o streaming de jogos.

---

## 3. Lidando com o Tráfego (Escalabilidade)

Para aguentar o pico de 1 milhão de usuários entrando ao mesmo tempo:

### O Primeiro Componente a Receber as Requisições:
* **O que usar:** **Balanceador de Carga (Load Balancer)**.

### Como ele evita a sobrecarga?
Ele é como um "recepcionista" inteligente:
1. Recebe *todas* as requisições dos usuários.
2. Ele monitora qual servidor de jogos está mais livre ou menos sobrecarregado.
3. Ele distribui cada novo usuário para o servidor mais "folgado", **garantindo que nenhum servidor pegue mais trabalho do que consegue aguentar**. (E ainda trabalha com o Auto Scaling para ligar mais servidores automaticamente se a fila de espera crescer).

---

## 4. Quem Fala com Quem? (Segurança)

Se o Serviço de Login precisar falar com o Serviço de Catálogo, ele não pode ser qualquer um.

### O Pilar de Segurança:
* **Conceito:** **Autorização de Serviço a Serviço (Service-to-Service IAM)**.

### Por que isso é essencial?
É a segurança interna:
* Dá uma "Identidade" e uma "Permissão" única para cada microsserviço.
* Isso garante que apenas o nosso Serviço de Login, com a chave correta, possa pedir informações ao Serviço de Catálogo. **Um serviço não pode se passar pelo outro!**


