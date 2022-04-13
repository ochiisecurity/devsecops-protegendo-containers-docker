# DevSecOps: protegendo containers docker

A Overture plataforma faz uso extensivo de Docker juntos tecnologia recipiente com Kubernetes. Ele nos permite facilmente abstrair os requisitos de tempo de execução dos diferentes componentes de back-end do sistema que são construídos utilizando diferentes linguagens de programação. Enquanto ainda permitindo-nos para dimensionar facilmente e obter as rotas de rede no lugar certo. Para se ter uma ideia, componentes de back-end no Overture podem ser escritos em NodeJS, Python ou Go. Nós escolher o idioma que se encaixa o trabalho. Desde que dependem fortemente de tecnologia Docker, nós também colocar uma grande quantidade de tempo para garantir esses recipientes. O que segue são alguns quick-wins em proteger seus recipientes.

## secure base

Entenda imagem que base que você está usando quando você iniciar a sua construção Docker de uma imagem base certa. Por exemplo, quando você quer Dockerize uma aplicação NodeJS e você começa a partir da imagem nó genérico, nota que uma distribuição total Debian é enviado no recipiente. Você não precisa a distribuição completa Debian em seu recipiente para obter a sua aplicação NodeJS ao trabalho. Considere usar as Alpine Linux imagens baseadas. Apenas o mínimo está incluído o que reduz a superfície de ataque do recipiente enquanto continua a fornecer um ambiente operacional decente. Para quase qualquer ambiente de tempo de execução da comunidade fornece uma variante Alpine. Por exemplo: nó: 8.16.0-alpino, pitão: 3.7.3-alpine3.8 e golang: 1.12.5-alpine3.9.

## reduza os privilégios do usuário

Por padrão, no caso do Dockerfile não especifica um usuário usando o usuário comando, o recipiente com base Linux será executado por padrão usando direitos de raiz. Não há um monte de casos de uso, onde um recipiente precisa executar usando permissões de root. A maioria dos casos você gostaria de restringir esses direitos de modo que no caso de um atacante recebe a execução de código no recipiente, ele não será executado com permissão raiz. Abaixo está um exemplo onde partimos de uma imagem Linux Alpine NodeJS seguro e criamos um novo grupo de usuários e adicionar um novo usuário a esse grupo. Por fim, transferir a propriedade usando o comando chown e mudar o contexto do usuário para awesomeuser e, finalmente, executar a aplicação NodeJS. Neste caso, quando um atacante consegue obter a execução de código por meio da aplicação NodeJS execução, seus direitos serão restritos e ele não será capaz de fazer muito dano.

```dockerfile
FROM node:8.16.0-alpine
RUN mkdir /app
RUN groupadd -r awesomegroup && useradd -r -s /bin/false -g awesomegroup awesomeuser
WORKDIR /app
COPY . /app
RUN chown -R awesomegroup:awesomeuser /app
USER awesomeuser
CMD node index.js
```

## destruir credenciais de acesso

Existem alguns casos em que segredos (por exemplo, tokens de acesso, senhas, certificados, chaves criptográficas, etc.) são necessários para extrair ou construir o código-fonte. Depois que essa etapa for concluída, as credenciais não serão mais necessárias e poderão ser excluídas com segurança. Isso pode ser feito explicitamente declarando um comando delete no Dockerfile ou usando contêineres intermediários que não estão mais disponíveis durante o tempo de execução. Isso garante que, caso o contêiner seja comprometido, as credenciais não estejam mais disponíveis.
Caso você precise de credenciais de acesso secretas durante o tempo de execução do aplicativo, não as armazene em código fixo ou como variáveis ​​de ambiente. Em vez disso, considere o uso de soluções semelhantes a cofres: Docker Swarm Secrets, Kubernetes Secrets, AWS Secrets etc.

## definir limites de recursos

Qualquer que seja o mecanismo de contêiner usado para executar o Docker, todos eles têm opções de configuração sobre a quantidade de recursos em termos de CPU e memória que o contêiner em execução pode usar. Certifique-se de que esses recursos estejam configurados de forma que fiquem restritos a um valor fixo que, quando atingido, ainda permita que o host opere em condições normais. Isso garante que, caso um hacker comprometa o contêiner, ele não possa esgotar o host de todos os seus recursos.

Por último, mas não menos importante, uma imagem segura do Docker é tão segura quanto o aplicativo que é executado nela. As práticas recomendadas de segurança de aplicativos ainda são a melhor medida para criar contêineres ou imagens seguras do Docker.
Pronto, vá e proteja seus contêineres Docker!
