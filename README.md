# **Aplicação Básica e Iniciante de BigData**

- ### _Como Configurar um Ambiente de Big Data no Ubuntu 24 LTS com Hadoop e Criar uma Aplicação Simples_
Este tutorial vai guiá-lo através do processo de instalação e configuração do **Apache Hadoop** no Ubuntu 24 LTS, além de ensinar como criar uma aplicação simples de Big Data: um clássico **Word Count** (Contagem de Palavras). Esse exemplo mostrará como o Hadoop processa grandes volumes de dados em paralelo usando o modelo **MapReduce**.

<br>1. **Pré-requisitos**
  - Ter um sistema Ubuntu 24 LTS.
  - Acesso à linha de comando (Terminal).
  - Conhecimento básico de Java (opcional, mas recomendado).

<br>2.  **Atualizar o sistema e instalar o Java**

Hadoop precisa do Java para funcionar, então a primeira coisa que faremos é garantir que o sistema está atualizado e que o Java está instalado.<br></br>

- **Atualize o sistema:** No terminal, execute o seguinte comando.

```
sudo apt update && sudo apt upgrade
```
<br></br>

- **Instale o Java (OpenJDK 11):** O Hadoop funciona bem com o *Java 11*, então vamos instalá-lo.

```
sudo apt install openjdk-11-jdk
```
<br></br>

- **Verifique a instalação do Java:** Certifique-se de que o Java foi instalado corretamente.

```
java -version
```

A saída deverá exibir algo como **"openjdk version 11.0.x"**.<br></br>

3. **Baixar e configurar o Hadoop**

Agora vamos instalar o *Hadoop*. Para este exemplo, utilizaremos a versão mais recente disponível.<br></br>

- **No terminal, execute o comando abaixo para baixar o *Hadoop*:**

```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
```
<br></br>

- **Extrair o arquivo baixado:** Descompacte o arquivo para ter acesso ao conteúdo do *Hadoop*.

```
tar -xvzf hadoop-3.3.6.tar.gz
```
<br></br>

- **Configurar as variáveis de ambiente:** Agora, precisamos configurar as variáveis de ambiente para que o sistema saiba onde encontrar os binários do *Hadoop*. Abra o arquivo *.bashrc* no editor de texto.

``nano ~/.bashrc`` ou ``vim ~/.bashrc``<br></br>
<br></br>

- **Caso você não tenha o editor de texto  "vim" instalado:** Abra o terminal e execute o comando.

```
sudo apt install vim
```
<br></br>

- **No final do arquivo, adicione as seguintes linhas:**

```
export HADOOP_HOME=/caminho/para/hadoop-3.3.6
export PATH=$PATH:$HADOOP_HOME/bin
```

Substitua ***/caminho/para/hadoop-3.3.6*** pelo diretório onde você extraiu o Hadoop (Exemplo:  /home/gustavo/hadoop-3.3.6). Em seguida, salve o arquivo e feche o editor.<br></br>

- **Também devemos adicionar as seguintes linhas ao final do arquivo, substituindo pelo caminho correto do Java:**

```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
export PATH=$PATH:$JAVA_HOME/bin
```
<br></br>

- **Atualizar as variáveis de ambiente:** Para aplicar as mudanças, execute o comando:

```
source ~/.bashrc
```
<br></br>

4. **Configurar o HDFS (Hadoop Distributed File System)**

O HDFS é o sistema de arquivos distribuído do Hadoop, onde você armazenará seus dados para processamento.<br></br>

- **Criar o diretório de entrada no HDFS:** Antes de rodar o exemplo de contagem de palavras, precisamos colocar o arquivo de entrada no HDFS. Primeiro, crie um diretório.

```
hdfs dfs -mkdir /input
```

***AVISO:*** O comando **"hdfs dfs -mkdir /input"** está criando um diretório dentro do sistema de arquivos distribuído do **Hadoop (HDFS)**, e não no seu sistema de arquivos local. Portanto, o caminho **/input** refere-se ao diretório no **HDFS**, que é o ambiente virtual onde o Hadoop gerencia seus dados.<br></br>

- **Criar um arquivo de entrada:** Vamos criar um arquivo de texto simples para usar no exemplo. No terminal, execute.

```
nano input.txt
```
<br></br>

- **Adicione o seguinte conteúdo ao arquivo:**

```
Hadoop is an open-source framework
Hadoop is widely used for Big Data
Hadoop processes large datasets
```

Salve e feche o arquivo.<br></br>

- **Enviar o arquivo para o HDFS:** Agora, coloque o arquivo no diretório de entrada do HDFS.

```
hdfs dfs -put input.txt /input
```
<br></br>

<br>5. **Criar a aplicação Word Count com Hadoop**

Agora que temos o arquivo de entrada no HDFS, vamos criar uma aplicação que conta a frequência de cada palavra usando o modelo MapReduce.<br></br>

- **Escrever o código Java para o Word Count:** Crie um arquivo Java para a aplicação de contagem de palavras.

``nano WordCount.java`` ou ``vim WordCount.java``<br></br>

- **Cole o seguinte código Java, que define o Mapper e o Reducer para o Hadoop:**

```
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {
    public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            String[] words = value.toString().split("\\s+");
            for (String w : words) {
                word.set(w);
                context.write(word, one);
            }
        }
    }

    public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```
<br></br>

- **Compilar o programa Java:** Compile o código WordCount.java com o Hadoop:

```
javac -classpath `hadoop classpath` -d . WordCount.java
```
<br></br>

- **Em seguida, crie o arquivo JAR do programa:**

```
jar -cvf wordcount.jar -C . .
```
<br></br>

<br>6. Executar o programa no Hadoop

Agora que o programa está pronto, vamos rodá-lo no Hadoop.<br></br>

- **Executar o programa Word Count:** Execute o programa, passando o diretório de entrada (/input) e o diretório de saída (/output).

```
hadoop jar wordcount.jar WordCount /input /output
```
<br></br>

- **Verificar os resultados:** Depois que o job terminar, visualize a saída com o seguinte comando.

```
hdfs dfs -cat /output/part-r-00000
```

Isso exibirá a contagem de cada palavra do arquivo **input.txt**.<br></br>

# Conclusão

Com este tutorial, você aprendeu a configurar o Hadoop no Ubuntu, a carregar dados no HDFS e a criar uma aplicação simples de contagem de palavras usando o modelo MapReduce. 

Esse exemplo pode ser expandido para processar conjuntos de dados maiores, e o mesmo conceito pode ser aplicado a problemas de Big Data mais complexos.

Agora você está pronto para mostrar essa aplicação para outras pessoas e ajudar a disseminar o conhecimento sobre Big Data!

Espero que esse tutorial seja útil para você e para outros interessados em aprender mais sobre Big Data! Se precisar de mais ajustes ou tiver dúvidas, estou à disposição.





















Você disse:
Consegui executar o comando: hdfs dfs -mkdir /input

Eu sou um aluno em fase de aprendizado, por isso tenho muitas dúvidas. Onde eu encontro esse diretório /input que eu criei? Gostaria de ver ele aqui na minha tela por curiosidade.
O ChatGPT disse:
ChatGPT
Ótimo, que bom que conseguiu criar o diretório no HDFS!

No entanto, o diretório /input que você criou não está no sistema de arquivos local do seu Ubuntu, mas sim no sistema de arquivos distribuído do Hadoop, o HDFS. Isso significa que você não pode acessá-lo diretamente como faria com diretórios no sistema local, como os encontrados em /home/.

Como visualizar o conteúdo do HDFS:
Para ver os diretórios e arquivos dentro do HDFS, você pode usar comandos do Hadoop. Vamos listar o conteúdo do diretório raiz do HDFS para você ver o diretório /input que acabou de criar.

Execute o seguinte comando:

bash
Copiar código
hdfs dfs -ls /
Este comando lista todos os diretórios e arquivos na raiz do HDFS. Você deve ver o diretório /input listado ali.

Para ver o conteúdo do diretório /input:
Se quiser ver o que está dentro do diretório /input, você pode usar o comando:

bash
Copiar código
hdfs dfs -ls /input
Isso vai mostrar o conteúdo do diretório que você criou, uma vez que você comece a carregar arquivos nele.

Curiosidade:
O HDFS é um sistema de arquivos separado que gerencia a distribuição e replicação de arquivos entre vários nós de um cluster Hadoop. Então, você sempre interage com ele através dos comandos hdfs dfs.

Se tiver mais dúvidas ou quiser explorar mais funcionalidades, pode perguntar!






Você disse:
Eu fiz esse comando para jogar o arquivo input.txt para dentro do diretório input do Hadoop correto? 

Mas ainda tem um input.txt no meu usuário root, eu posso excluir ele?
O ChatGPT disse:
ChatGPT
Sim, você está correto! Quando você executou o comando:

bash
Copiar código
hdfs dfs -put input.txt /input
Isso copiou o arquivo input.txt do seu sistema de arquivos local (que estava no diretório do root) para o diretório /input no HDFS. O arquivo agora está dentro do Hadoop Distributed File System (HDFS).

Sobre o arquivo no seu sistema local:
Agora que o arquivo input.txt já foi transferido para o HDFS, você pode excluí-lo do diretório local, se não precisar mais dele. O arquivo que está no HDFS não será afetado.

Para excluir o arquivo do seu diretório local (no sistema de arquivos do Linux), você pode usar o comando:

bash
Copiar código
rm input.txt
Isso vai remover o arquivo input.txt do seu sistema local, mas o arquivo continuará seguro no HDFS.






Você disse:
Você pediu para eu executar o seguinte comando: nano WordCount.java

Eu executarei o comando vim WordCount.java pois eu prefiro o vim do que o nano.

Esse arquivo será criado dentro do Hadoop também? Ou será dentro do meu sistema local? 
O ChatGPT disse:
ChatGPT
Memória atualizada
Quando você criar o arquivo WordCount.java com o comando vim WordCount.java, esse arquivo será criado no seu sistema de arquivos local, e não no Hadoop (HDFS). Ou seja, ele estará no diretório em que você estiver no momento em que executar o comando.

O Hadoop executa o código Java a partir do sistema local, mas os dados que ele processa (como o arquivo input.txt) estão no HDFS. O fluxo é assim:

Criação e compilação do código Java — isso acontece no seu sistema de arquivos local.
Execução do código Hadoop (MapReduce) — o Hadoop processa os dados armazenados no HDFS com base no código Java que você criou localmente.
Então, ao criar WordCount.java usando vim, você está criando esse arquivo no diretório local.






Você disse:
Você me passou um código Java que define o Mapper e o Reducer para o Hadoop. O que seriam essas coisas? Porque precisamos delas? Poderia me explicar o código que você criou?
O ChatGPT disse:
ChatGPT
Claro! O código Java que define o Mapper e o Reducer é fundamental para a arquitetura do Hadoop e do seu modelo de programação MapReduce. Vamos entender o que cada um faz e, em seguida, eu explicarei o código que você viu.

O que é MapReduce?
MapReduce é um modelo de programação que permite processar grandes volumes de dados em paralelo. Ele divide o trabalho em duas etapas principais:

Map (Mapper): Esta etapa envolve o processamento inicial dos dados. O Mapper pega os dados de entrada e os transforma em pares de chave-valor. Por exemplo, em um programa de contagem de palavras, o Mapper recebe linhas de texto e emite pares onde a chave é uma palavra e o valor é o número 1.

Reduce (Reducer): Após a etapa de mapeamento, os pares de chave-valor gerados pelos Mappers são agrupados por chave. O Reducer pega esses pares e os processa para produzir uma saída consolidada. No exemplo de contagem de palavras, o Reducer soma todos os valores (números 1) para cada palavra.

Estrutura do Código
Aqui está um breve resumo do código que você tem para o Word Count:

java
Copiar código
import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
Importações: Importamos as classes necessárias para o funcionamento do Hadoop e para a manipulação de dados.
Mapper
java
Copiar código
public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
        String[] words = value.toString().split("\\s+");
        for (String w : words) {
            word.set(w);
            context.write(word, one);
        }
    }
}
TokenizerMapper: Esta classe estende a classe Mapper.
map: O método map é onde a lógica de mapeamento ocorre. Recebe a entrada (uma linha de texto) e divide essa linha em palavras.
String[] words = value.toString().split("\\s+");: Essa linha divide a linha de texto em um array de palavras com base nos espaços.
context.write(word, one);: Para cada palavra, emite um par chave-valor onde a chave é a palavra e o valor é 1 (indicando que a palavra foi encontrada uma vez).
Reducer
java
Copiar código
public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        result.set(sum);
        context.write(key, result);
    }
}
IntSumReducer: Esta classe estende a classe Reducer.
reduce: O método reduce é onde a lógica de redução ocorre. Recebe pares de chave-valor e soma os valores para cada chave.
int sum = 0;: Inicializa uma variável para armazenar a soma.
for (IntWritable val : values): Itera sobre os valores associados à chave (todas as ocorrências da palavra).
result.set(sum);: Armazena a soma total para a palavra e emite o resultado.
Main
java
Copiar código
public static void main(String[] args) throws Exception {
    Configuration conf = new Configuration();
    Job job = Job.getInstance(conf, "word count");
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, new Path(args[0]));
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
    System.exit(job.waitForCompletion(true) ? 0 : 1);
}
main: O ponto de entrada da aplicação. Aqui, configuramos o job Hadoop, definimos as classes Mapper e Reducer, e configuramos as entradas e saídas do job.
job.setMapperClass(TokenizerMapper.class);: Define a classe Mapper que será utilizada.
job.setReducerClass(IntSumReducer.class);: Define a classe Reducer que será utilizada.
FileInputFormat.addInputPath(...): Define o caminho do arquivo de entrada.
FileOutputFormat.setOutputPath(...): Define onde os resultados da execução serão armazenados.
Resumo
Mapper: Divide a entrada em pares de chave-valor (palavra e contagem).
Reducer: Agrupa esses pares por chave e agrega (soma) os valores.
Essas duas etapas permitem que o Hadoop processe dados massivos de forma distribuída, fazendo uso eficaz de recursos computacionais. Se você tiver mais perguntas ou precisar de mais detalhes, fique à vontade para perguntar!






Você disse:
Você pediu para eu executar o seguinte comando: javac -classpath hadoop classpath -d . WordCount.java

Que seria para o Hadoop ler o código que criamos e guarda-lo correto?

Depois você pediu para eu rodar esse comando: jar -cvf wordcount.jar -C . .

Que seria para transformar o arquivo que está no Hadoop em um executável Java, que seria o Jar correto?
O ChatGPT disse:
ChatGPT
Exatamente! Vamos detalhar cada um desses comandos para esclarecer melhor:

Compilação do Código Java
bash
Copiar código
javac -classpath `hadoop classpath` -d . WordCount.java
javac: Este é o compilador Java, que transforma o código fonte Java (neste caso, WordCount.java) em bytecode que pode ser executado pela Java Virtual Machine (JVM).
-classpath hadoop classpath: O parâmetro -classpath especifica onde o compilador deve procurar as classes necessárias. O comando hadoop classpath fornece todos os caminhos de bibliotecas do Hadoop que você precisa para compilar seu código. Assim, ele garante que o código tenha acesso a todas as classes do Hadoop necessárias durante a compilação.
-d .: O parâmetro -d especifica o diretório de saída onde os arquivos compilados devem ser armazenados. O ponto . indica que os arquivos compilados devem ser armazenados no diretório atual.
WordCount.java: Este é o arquivo que está sendo compilado.
Criação do JAR Executável
bash
Copiar código
jar -cvf wordcount.jar -C . .
jar: Este é um utilitário para criar arquivos JAR (Java Archive). Um arquivo JAR é uma forma de empacotar vários arquivos em um único arquivo, que pode ser executado na JVM.
-cvf: Esses parâmetros significam:
c: Criar um novo arquivo JAR.
v: Exibir informações detalhadas durante o processo de criação (verbose).
f: Especificar o nome do arquivo JAR que será criado (neste caso, wordcount.jar).
-C . .: O parâmetro -C muda o diretório para o especificado (aqui, . significa o diretório atual) e adiciona todos os arquivos encontrados neste diretório ao JAR. O segundo ponto . é para incluir todos os arquivos e subdiretórios do diretório atual.
Resumo
O primeiro comando compila seu código Java e o prepara para execução, armazenando os arquivos compilados no diretório atual.
O segundo comando empacota esses arquivos compilados em um arquivo JAR executável, que você pode rodar no Hadoop.
Após executar esses comandos, você terá um arquivo wordcount.jar que pode ser executado no Hadoop, permitindo que você processe os dados de acordo com a lógica definida no seu código. Se você tiver mais perguntas, estou aqui para ajudar!






Você disse:
Você pediu para eu executar esse comando: hadoop jar wordcount.jar WordCount /input /output

Ele executou o código que você criou que seria o wordcount.jar e gerou o resultado /output correto?

Esse comando hdfs dfs -cat /output/part-r-00000 me mostra o resultado que gerou. Entretanto não entendi o resultado que gerou, poderia me explicar o resultado e qual o objetivo da aplicação BigData que você fez?

root@rarysson-B550M-H:~# hdfs dfs -cat /output/part-r-00000
Big	1
Data	1
Hadoop	3
an	1
datasets	1
for	1
framework	1
is	2
large	1
open-source	1
processes	1
used	1
widely	1
O ChatGPT disse:
ChatGPT
Exatamente! Vamos revisar os pontos que você mencionou e analisar o resultado gerado pela sua aplicação de contagem de palavras no Hadoop.

Comando Executado
O comando que você executou:

bash
Copiar código
hadoop jar wordcount.jar WordCount /input /output
hadoop jar: Este comando inicia a execução de um arquivo JAR que contém seu código Java compilado.
wordcount.jar: Este é o arquivo JAR que você criou, contendo a lógica para contar palavras.
WordCount: Este é o nome da classe principal que você definiu no seu código Java.
/input: Este é o diretório no HDFS onde está o arquivo de entrada (input.txt) que você criou.
/output: Este é o diretório onde o Hadoop irá armazenar os resultados da execução do programa.
Resultado do Comando hdfs dfs -cat /output/part-r-00000
O comando que você usou para visualizar o resultado:

bash
Copiar código
hdfs dfs -cat /output/part-r-00000
Este comando mostra o conteúdo do arquivo de saída gerado pelo seu programa. O que você obteve é uma contagem de palavras, onde cada linha representa uma palavra do arquivo de entrada e o número correspondente de vezes que essa palavra aparece.

Explicação do Resultado
Aqui está o resultado que você obteve:

kotlin
Copiar código
Big	1
Data	1
Hadoop	3
an	1
datasets	1
for	1
framework	1
is	2
large	1
open-source	1
processes	1
used	1
widely	1
Palavras: Cada palavra é listada na primeira coluna.
Contagem: O número à direita de cada palavra indica quantas vezes essa palavra aparece no texto que você processou.
Objetivo da Aplicação Big Data
O objetivo dessa aplicação Big Data é demonstrar a capacidade do Hadoop de processar e analisar grandes volumes de dados de forma distribuída. O exemplo da contagem de palavras é um clássico no aprendizado sobre Big Data porque:

Simplicidade: É fácil de entender e implementar.
Fundamentos do MapReduce: Introduz os conceitos de Map e Reduce:
Map: O Mapper divide a entrada (neste caso, o texto) em palavras e emite pares (palavra, 1).
Reduce: O Reducer pega todos os pares gerados pelo Mapper, agrupa-os pela palavra e soma os valores associados a cada palavra, resultando na contagem total.
Essa aplicação serve como uma base para compreender como sistemas de Big Data podem lidar com dados em larga escala e realizar análises úteis. Com esse entendimento, você pode explorar outros casos de uso mais complexos e variados no contexto de Big Data. Se você tiver mais perguntas ou quiser explorar mais sobre o Hadoop ou Big Data, fique à vontade para perguntar!






Você disse:
Você disse que o número à direita de cada palavra indica quantas vezes essa palavra aparece no texto que você processou. Onde está esse texto? Eu posso altera-lo e escrever outro texto no lugar para eu poder executar ele? Posso fazer outros testes?
O ChatGPT disse:
ChatGPT
Sim, você pode alterar o texto que está no arquivo input.txt e realizar novos testes! O texto original que você criou estava assim:

kotlin
Copiar código
Hadoop is an open-source framework
Hadoop is widely used for Big Data
Hadoop processes large datasets
Para Alterar o Texto
Abra o arquivo input.txt para edição usando o vim (ou outro editor de sua preferência):

bash
Copiar código
vim input.txt
Altere o conteúdo do arquivo conforme sua necessidade. Você pode adicionar novas linhas, modificar o texto existente ou até mesmo remover algumas linhas.

Exemplo de novo texto:

kotlin
Copiar código
Big Data is transforming industries.
Hadoop makes it easy to process large datasets.
Learning Hadoop can be fun and insightful.
Salve e feche o arquivo. No vim, você pode fazer isso pressionando Esc, digitando :wq, e pressionando Enter.

Depois de Alterar o Texto
Após atualizar o arquivo input.txt, você precisa repetir o processo de contagem de palavras para ver os novos resultados:

Coloque o arquivo no HDFS novamente (apenas se você tiver removido o antigo):

bash
Copiar código
hdfs dfs -put input.txt /input
Execute o seu programa de contagem de palavras novamente:

bash
Copiar código
hadoop jar wordcount.jar WordCount /input /output
Verifique o resultado:

bash
Copiar código
hdfs dfs -cat /output/part-r-00000
Observações
Vários Testes: Você pode fazer vários testes, alterando o texto no input.txt, executando o programa e visualizando os resultados quantas vezes quiser.

Limpar Resultados Anteriores: Se você deseja executar testes com diferentes textos, pode ser uma boa ideia excluir os diretórios de saída anteriores. Para fazer isso, você pode usar o seguinte comando antes de executar seu código novamente:

bash
Copiar código
hdfs dfs -rm -r /output
Isso removerá o diretório /output existente, permitindo que você crie um novo diretório de saída ao executar o programa novamente.

Sinta-se à vontade para experimentar! Se tiver mais perguntas ou precisar de ajuda, estou aqui para ajudar!


