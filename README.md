

---

# README - Monitoramento do Jenkins com Prometheus e Grafana

Seguindo os passos apresentados neste repositório, conseguimos configurar o Jenkins em um servidor Tomcat e acompanhamos as métricas para monitoramento de seus dados com o **Prometheus**, que irá coletar essas métricas, e o **Grafana**, que as apresentará de forma clara e intuitiva. Todos os passos serão feitos utilizando containers Docker.

## 1. Baixar o Jenkins

Primeiro, vamos baixar o arquivo **WAR** do Jenkins para o nosso host:

```bash
curl -L -o jenkins.war https://get.jenkins.io/war-stable/latest/jenkins.war
```

## 2. Configurar o Tomcat

Agora, iremos criar um container Docker com o **Tomcat**:

```bash
docker run -d --name tomcat-container -p 8080:8080 tomcat:jdk17
```

**Nota**: Lembre-se de que iremos acessar a interface do Jenkins através da porta **8080**.

Verifique se o container está **UP**:

```bash
docker ps
```

---

## 3. Acessar o Jenkins e Recuperar a Senha Inicial

Após iniciar o Tomcat, acesse o Jenkins em:

```
http://localhost:8080/jenkins
```

O Jenkins irá solicitar uma senha, que é a senha do administrador. Para obtê-la, siga os seguintes passos:

![solicitando senha de acesso](img/login.jpeg)


```bash
docker exec -it tomcat-container bash
cd /root/.jenkins/secrets/
cat initialAdminPassword
```

Essa senha será usada para fazer o login no Jenkins pela primeira vez.

---

## 4. Instalar Extensões sugeridas no Jenkins e crie um usuário

Instale as extensões sugeridas:

![instalando extensões](img/pluguin.jpeg)
![processo de instalação](img/processo.jpeg)

agora crie um usuário:

![processo de instalação](img/user.jpeg)

Como não pretendemos acessar o Jenkins de forma remota avance sem alterar a URL:

![processo de instalação](img/url.jpeg)

Caso a tela principal do Jenkins apareça em diante, parabéns você conseguiu:

![processo de instalação](img/tela.jpeg)

- Agora vamos adicionar o pluguin do Prometheus, respnsável pela integração com a ferramenta, vá até **Gerenciar Jenkins > Gerenciar Plugins**.
- Na aba **Disponíveis**, procure por **Prometheus Metrics Plugin**.
- Instale o plugin e reinicie o container do Tomcat:

![instalação prometheus](img/instalprometheus.jpeg)

```bash
docker restart tomcat-container
```



## 5. Configurar o Plugin Prometheus no Jenkins

1. Faça logins no Jenkins. Vá até **Gerenciar Jenkins > Configurar o Sistema**.
2. Na seção **Prometheus**, configure as opções conforme o exemplo abaixo:

    - **Path**: `/prometheus` (padrão)
    - **Namespace**: `default`
    - **Enable authentication**: **Desabilitado** (opcional, se o Jenkins for exposto publicamente)
    - **Collecting metrics period**: `120` (2 minutos)
    - **Job attribute name**: `jenkins_job`

![configuração de parametros prometheus](img/confprometeus.jpeg)

Clique em **Salvar**.

---

## 7. Configurar o Prometheus

### Criar o arquivo `prometheus.yml`

1. Crie o arquivo **`prometheus.yml`** no diretório do seu projeto com o seguinte conteúdo:

    ```yaml
    global:
      scrape_interval: 10s  # Coleta de métricas a cada 10 segundos

    scrape_configs:
      - job_name: 'jenkins'  # Monitorar apenas Jenkins
        metrics_path: '/jenkins/prometheus'  # Endpoint do Jenkins Metrics Plugin
        static_configs:
          - targets:
              - '172.17.0.2:8080'  # Substitua pelo IP do container Jenkins/Tomcat
    ```

2. Envie o arquivo de configuração para o container do Prometheus:

    ```bash
    docker cp prometheus.yml prometheus:/etc/prometheus/prometheus.yml
    ```

3. Reinicie o container do Prometheus para aplicar as configurações:

    ```bash
    docker restart prometheus
    ```

4. Acesse o Prometheus em `http://localhost:9090/targets` e verifique se o endpoint do Jenkins está **UP**.

![EndopointUP](img/endup.jpeg)

---

## 8. Configurar o Grafana

1. Crie o container do Grafana:

    ```bash
    docker run -d --name grafana_prometheus -p 3000:3000 grafana/grafana:latest
    ```

2. Acesse o Grafana em `http://localhost:3000/login` com o login padrão:
   - **Usuário**: `admin`
   - **Senha**: `admin`

Após o login, defina uma nova senha.

![grafanalogin](img/graf.jpeg)

3. Conectar o Grafana ao Prometheus:
    - Vá para **Configuration > Data Sources** e selecione **Prometheus**.
    - Insira a URL do Prometheus: `http://172.17.0.3:9090` (substitua pelo IP correto do container Prometheus).
    ```bash
    docker network inspect bridge 
    ```

    - Clique em **Save & Test**.

---

## 9. Criar um Dashboard no Grafana

1. No Grafana, acesse **Dashboards > New Dashboard**.
2. Clique em **Add new panel**.
3. Selecione a fonte de dados **Prometheus**.
4. Na consulta, insira uma das métricas, como `default_jenkins_uptime`, e clique em **Run Query**.
5. Escolha a visualização adequada (como **Stat** ou **Time series**).
6. Salve o painel e o dashboard.

---

## 10. Refinar a Exibição no Grafana

- No painel, configure as opções de visualização para exibir as métricas de forma mais clara.
- Ajuste os tipos de gráficos conforme a necessidade (por exemplo, gráficos de barras, séries temporais, etc.).
- Deixei o Json do Dashboard do grafana neste repositório.

![grafanalogin](img/dashgraf.jpeg)

---

## 11. Testar e Validar

1. Acesse o dashboard no Grafana e verifique se os dados estão sendo exibidos corretamente.
2. Se os dados não aparecerem, revise as configurações do Prometheus e Grafana, garantindo que as consultas estão corretas.

---

## 12. Conclusão

Agora nós temos um ambiente configurado para monitorar o Jenkins usando Prometheus e Grafana, tudo em containers Docker. 

01000100 01100101 01110101 01110011 00100000 11101001 00100000 01100110 01101001 01100101 01101100 00100001


---

