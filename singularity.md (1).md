### Tutorial Singularity
  
##### Criação: Gustavo - 14/08/2023  
##### Última atualização: Gustavo, ChatGPT (chat.openai.com), Bing (bing.com) e Bard (bard.google.com) - 14/08/2023 
##### Esboço: Documento em revisão
  
#### Objetivo
O *Singularity* é um software que permite criar e executar contêineres em sistemas Linux. Usando o *Singularity*, é possível criar um contêiner no seu próprio desktop ou servidor contendo o(s) software(s) que você quer usar no CENAPAD-SP. Depois de criado o contêiner, você terá um arquivo `.sif` com todo o necessário para transferir e executar no CENAPAD-SP. Este processo facilita a instalação de softwares complexos e com muitas dependências sem necessidade de permissões de administrador (root), além de garantir maior reprodutibilidade.

  ---
#### Roteiro de instalação no Ubuntu 22.04 
*Também funciona com Ubuntu via WSL2 do Windows 10 ou 11 e em outras distribuições baseadas em Debian* 

 1. **Baixe o singularity**

Baixe o arquivo singularity-container_3.8.7_amd64.deb da página de releases no repositório GitHub do singularity - https://github.com/apptainer/singularity/releases
```bash
cd /tmp
wget https://github.com/apptainer/singularity/releases/download/v3.8.7/singularity-container_3.8.7_amd64.deb
```

2.  **Instale o pacote deb com os utilitários apt ou dpkg**

Com apt:
```bash
sudo apt install ./singularity-container_3.8.7_amd64.deb
```
Com dpkg:
```bash
sudo dpkg -i singularity-container_3.8.7_amd64.deb
```
 3. **Verifique se o Singularity está instalado corretamente**
 
```bash
singularity version
```
Você deve ver a saída:
```bash
3.8.7
```
---
#### Preparação de imagem do Singularity  

 4. **Prepare uma imagem do Singularity**

Você pode criar uma imagem do *Singularity* a partir de um arquivo de definição (.def), que especifica os componentes e configurações do contêiner. Você pode encontrar alguns exemplos de arquivos de definição na documentação do Singularity. Por exemplo, para criar uma imagem baseada no Ubuntu 22.04 com alguns pacotes básicos, crie um arquivo chamado `ubuntu.def` com o seguinte conteúdo:
```bash
Bootstrap: docker
From: ubuntu:22.04
%post
apt-get -y update
apt-get -y install wget git vim
%environment
export LC_ALL=C
export PATH=/usr/local/bin:$PATH
%runscript
exec /bin/bash "$@"
```
> No exemplo acima, a seção `%post` indica os comandos que serão executados na base Ubuntu. Instale todos os pacotes que serão necessários para seu software rodar com apt ou outros utilitários. A seção `%environment` indica as variáveis de ambiente necessárias. Por fim, a seção runscript indica como um container baseado nesta imagem será executado. O comando `exec /bin/bash "$@"` indica que, por padrão, será executado um shell bash quando o contêiner for executado. 

 5. **Construa a imagem do Singularity usando o comando build**

Você precisa ter privilégios de root para construir uma imagem em formato *squashfs (sif)*, que é o formato recomendado para imagens portáteis e compactas. Execute o seguinte comando para construir a imagem chamada ubuntu.sif a partir do arquivo ubuntu.def:
```bash
sudo singularity build ubuntu.sif ubuntu.def
```
6. **Execute a imagem do Singularity usando o comando run ou exec**

Você pode executar a imagem do *Singularity* sem privilégios de root, desde que tenha permissão de leitura na imagem. O comando run irá executar o script definido na seção `%runscript` do arquivo de definição, que neste caso é um shell bash interativo. O comando exec irá executar qualquer comando que você especificar dentro do contêiner. Por exemplo, para executar um shell bash interativo dentro da imagem ubuntu.sif, execute:
```bash
singularity run ubuntu.sif
```
Para executar o comando ls dentro da imagem ubuntu.sif, execute:
```bash
singularity exec ubuntu.sif ls
```
---
#### Transferência e Execução do contêiner singularity no ambiente Lovelace

7. **Transfira a imagem do contêiner**

Copie a imagem para algum subdiretório no seu home, no ambiente lovelace. Use algum caminho de diretório existente criado anteriormente.
```bash
scp -P 31459 ubuntu.sif meu_username@cenapad.unicamp.br:~/homelovelace/imagens_singularity/
```
8. **Teste sua imagem**

Acesse a máquina Lovelace, carregue o módulo singularity e faça um teste rápido da sua imagem.
```bash
ssh -p 31459 meu_username@cenapad.unicamp.br
ssh lovelace
module load singularity
singularity exec ubuntu.sif ls
```

> Se tudo der certo, você verá os arquivos do seu homedir.

9. **Prepare o script de job PBS**

Agora é hora de preparar o seu script de job *PBS*. Escolha a fila e configure os parâmetros de acordo com as informações do [Guia do Usuário - Ambiente Lovelace - Filas e Limites](https://www.cenapad.unicamp.br/parque-computacional/equipamentos/dell-lovelace/execucao-jobs.shtml#jobslimites)

Há exemplos de scripts de job com o Singularity em [Guia do Usuário - Ambiente Lovelace - Singularity](
https://www.cenapad.unicamp.br/parque-computacional/equipamentos/dell-lovelace/singularity.shtml#singularity)

De modo geral, no lugar em que você executaria seu software diretamente, use-o através do *Singularity*. Por exemplo, em vez de:
```
<BINARIO_DO_MEU_SOFTWARE> <PARAMETROS>
```
Use:
```bash
module load singularity
singularity exec ubuntu.sif <BINARIO_DO_MEU_SOFTWARE> <PARAMETROS>
```
Por padrão, o *Singularity* o seu diretório home é mapeado para dentro do contêiner *Singularity*. Então basta indicar o caminho dos inputs e outputs como usual, se necessário.

 - **Submeta seu job**
SUbmeta o job com o comando `qsub <nome_do_script_do_job>` e aguarde a execução. Monitore o seu job com o comando `qstat <job_id>`. Veja mais informações sobre os comandos do PBS no [Guia do Usuário - Ambiente Lovelace - Execução de jobs](https://www.cenapad.unicamp.br/parque-computacional/equipamentos/dell-lovelace/execucao-jobs.shtml#jobs)
```bash
qsub <arquivo_do_seu_script_de_job>
qstat <job_id>
```
---
#### Referências
-  [Quick Start — Singularity User Guide 3.8 documentation - Apptainer](https://apptainer.org/user-docs/master/quick_start.html  )
- [Introduction to Singularity — Singularity container 3.8 - Sylabs](https://docs.sylabs.io/guides/3.8/user-guide/introduction.html)
