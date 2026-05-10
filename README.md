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

## Enumeração de Serviços Disponíveis

Foi utilizado o **Nmap** para identificar serviços rodando no alvo:
```bash
nmap -sV 10.0.0.20
```
### Principais serviços identificados:
- Porta 21/tcp: FTP (vsftpd 2.3.4)
- Portas 139/445/tcp: SMB (Samba smbd 3.X)
Esses serviços são frequentemente alvos de ataques de força bruta e password spraying devido à sua exposição e configuração insegura.

---

## Criação de lista de usuários e lista de senhas

Para os testes, foram utilizadas listas de usuários e senhas.

### Lista de Usuários

A lista de usuários foi gerada a partir da **enumeração de contas existentes no serviço SMB** do Metasploitable, utilizando a ferramenta `enum4linux`.
```bash
enum4linux -U 10.0.0.20 | grep "user:" | cut -d '[' -f2 | cut -d ']' -f1 > users.txt
```
Esse comando realiza a enumeração de usuários do serviço SMB do alvo e extrai apenas os nomes de usuário, salvando-os em users.txt.

### Lista de senhas

Utilizamos algumas senhas simples e a senha padrão do Metasploitable através do comando abaixo:
```bash
cat << EOF > passwords.txt
msfadmin
password
123456
root
admin
EOF
```
Esse comando captura tudo o que for digitado no terminal até digitarmos **EOF**.

---

## Ataques realizados

### 1. Força Bruta em FTP
Comando utilizado: 
```bash
medusa -h 10.0.0.20 -U users.txt -P passwords.txt -M ftp
```
O ataque busca identificar credenciais válidas para o serviço FTP rodando no Metasploitable 2.
> Como estamos em ambiente de laboratório, é possível utilizar a flag `-t` ao final do comando para acelerar a execução. O número padrão de **threads** é 16, mas podemos forçar para 32 ou 64.

### 2. Password Spraying
Comando utilizado:
```bash
medusa -h 10.0.0.20 -U users.txt -P passwords.txt -M smbnt
```
Este ataque testa combinações de usuários e senhas no serviço SMB, buscando acessos não autorizados e enumerando usuários válidos.

---

## Medidas de Prevenção e Hardening
Para mitigar os riscos de ataques de força bruta (FTP) e enumeração/spraying (SMB) demonstrados neste laboratório, as seguintes práticas de segurança são recomendadas:

### 1. Fortalecimento de Credenciais (Password Policy)
A primeira linha de defesa contra ataques de força bruta é a adoção de uma política de senhas robusta:

- **Complexidade e Entropia:** Utilizar senhas com no mínimo 12 caracteres, combinando maiúsculas, minúsculas, números e símbolos. Isso aumenta exponencialmente o tempo necessário para um ataque de força bruta ser bem-sucedido. (ex: G7a!pL9z@vQ2)
- **MFA (Multi-Factor Authentication):** Implementar o segundo fator de autenticação sempre que possível. Mesmo que o atacante descubra a senha, ele não conseguirá acessar o serviço sem o código temporário.
- **Bloqueio de Conta (Account Lockout):** Configurar o sistema para bloquear temporariamente o usuário após 3 ou 5 tentativas falhas. Isso inviabiliza ataques de dicionário em larga escala.

### 2. Configuração de Firewall
Um **firewall bem configurado** limita o acesso aos serviços, reduzindo a exposição a ataques externos:

- **Segregação de Rede:** Serviços sensíveis como SMB e bancos de dados nunca devem estar expostos diretamente à internet. Devem ser acessíveis apenas via **VPN*** ou dentro de uma **VLAN** protegida.
- **Default Deny:** Configurar o firewall para bloquear todo o tráfego de entrada por padrão e liberar apenas as portas estritamente necessárias (ex: porta 80/443 para web).
- **Whitelist de IPs:** Para serviços administrativos como FTP ou SSH, permitir conexões apenas de IPs conhecidos e confiáveis.
- Atualize regras regularmente conforme mudanças no ambiente.
### 3. Outras recomendações
- **Desativação de Protocolos legados:** Desabilitar o SMBv1 (altamente vulnerável) e priorizar versões mais seguras como SMBv3 com assinatura de pacotes. Para transferência de arquivos, substituir o FTP (que trafega dados em texto claro) por **SFTP** ou **FTPS**.
- **Fail2Ban:** Implementar ferramentas de prevenção de intrusão que monitoram os logs em tempo real e banem automaticamente, via firewall, IPs que apresentarem comportamento suspeito de força bruta.
- **Desativação de Contas Padrão:** Renomear usuários administradores (admin, root) e remover contas de demonstração que acompanham instalações padrão de softwares.
- **Gestão de Patches e Atualizações Críticas:** Manter o Kernel do Sistema Operacional e todos os pacotes de serviços (como vsftpd ou samba) rigorosamente atualizados. A maioria dos ataques exploram vulnerabilidades conhecidas (CVEs) para as quais já existem correções.
---

## Observações

- Todos os testes foram realizados em ambiente controlado e isolado.
- O Metasploitable 2 é uma máquina intencionalmente vulnerável, destinada exclusivamente para fins educacionais.
- Os resultados dos ataques devem ser utilizados para reforçar políticas de segurança, conscientização e monitoramento de sistemas.

---

<div align="center">
    Desenvolvido por <strong><a href="https://www.linkedin.com/in/patryck-pereira-5104a6140/">Patryck Pereira</a></strong>.<br>
    Projeto parte do Bootcamp de Cibersegurança da DIO.
    <br><br>
    <a href="https://web.dio.me/home">
        <img src="https://img.shields.io/badge/Desafio-DIO-0077B6?style=for-the-badge" alt="Desafio DIO Badge">
    </a>
</div>
