# Laboratório de Testes de Penetração: Ataques de Força Bruta

## Objetivos do Lab

O objetivo deste laboratório é simular ataques de força bruta e password spraying em serviços vulneráveis (FTP e SMB) utilizando o Kali Linux como plataforma de ataque e o Metasploitable 2 como alvo. Os testes visam:

- Demonstrar a vulnerabilidade de autenticação em serviços de rede.
- Praticar técnicas de enumeração e ataque de força bruta.
- Documentar o processo para fins educacionais e de auditoria.

---

## Configuração de Rede das VMs

As VMs utilizadas neste laboratório foram configuradas em uma rede interna privada (`dio-kalilab`) para garantir isolamento e segurança durante os testes.

### Kali Linux (Atacante)
- **IP:** `10.0.0.10/24`
- **Interface:** `eth0`
- **Comando utilizado:**
  ```bash
  sudo ip addr add 10.0.0.10/24 dev eth0
  sudo ip link set eth0 up
  ```

### Metasploitable 2 (Alvo)

- **IP:** `10.0.0.20/24`
- **Interface:** `eth0`
- **Comando utilizado:**
    ```bash
    sudo ip addr add 10.0.0.20/24 dev eth0
    sudo ip link set eth0 up
    ```

---



