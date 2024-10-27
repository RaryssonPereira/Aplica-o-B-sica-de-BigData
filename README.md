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


<br>3. **Baixar e configurar o Hadoop**

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


<br>4. **Configurar o HDFS (Hadoop Distributed File System)**

O HDFS é o sistema de arquivos distribuído do Hadoop, onde você armazenará seus dados para processamento.<br></br>

- **Criar o diretório de entrada no HDFS:** Antes de rodar o exemplo de contagem de palavras, precisamos colocar o arquivo de entrada no HDFS. Primeiro, crie um diretório.

```
hdfs dfs -mkdir /input
```

***AVISO:*** O comando **"hdfs dfs -mkdir /input"** está criando um diretório dentro do sistema de arquivos distribuído do **Hadoop (HDFS)**, e não no seu sistema de arquivos local. Portanto, o caminho ***/input*** refere-se ao diretório no **HDFS**, que é o ambiente virtual onde o Hadoop gerencia seus dados.<br></br>


- **Para ver os diretórios e arquivos dentro do HDFS:** Você pode usar comandos do Hadoop.

```
hdfs dfs -ls /
```

Este comando lista todos os diretórios e arquivos na raiz do **HDFS**. Você deve ver o diretório ***/input*** listado ali.<br></br>


- **Criar um arquivo de entrada:** Vamos criar um arquivo de texto simples para usar no exemplo. No terminal, execute.

``nano input.txt`` ou ``vim input.txt``
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

Você copiou o arquivo ***input.txt*** do seu sistema de arquivos local (que estava no diretório do root) para o diretório ***/input*** no dentro do **Hadoop Distributed File System (HDFS)**.<br></br>


- **Se quiser ver o que está dentro do diretório /input, você pode usar o comando:**

```
hdfs dfs -ls /input
```

Isso vai mostrar o conteúdo do diretório que você criou, uma vez que você comece a carregar arquivos nele.<br></br>


- **Agora que o arquivo input.txt já foi transferido para o HDFS:** Você pode excluí-lo do diretório local, se não precisar mais dele:

```
rm input.txt
```

Isso vai remover o arquivo ***input.txt*** do seu sistema local, mas o arquivo continuará seguro no **HDFS**.<br></br>


<br>5. **Criar a aplicação Word Count com Hadoop**

Agora que temos o arquivo de entrada no HDFS, vamos criar uma aplicação que conta a frequência de cada palavra usando o modelo MapReduce.<br></br>

- **Criando um Diretório para o Word Count:** Vamos criar um diretório para colocar nosso arquivo executável, assim ficará mais organizado e facilitará caso você queira modificar o Script.

```
mkdir bigdata_projeto
```

Depois de ter criado o diretório, acesse ele.

```
cd bigdata_projeto/
```

***AVISO:*** Você pode escolher o nome que dará para o diretório, escolhi **bigdata_projeto** apenas como exemplo.<br></br>


- **Escrever o código Java para o Word Count:** Crie um arquivo Java para a aplicação de contagem de palavras.

``nano WordCount.java`` ou ``vim WordCount.java``

***AVISO:*** Quando você criar o arquivo **WordCount.java**, esse arquivo será criado no diretório que você criou, e não no **Hadoop (HDFS)**. Ou seja, ele estará no diretório em que você estiver no momento em que executar o comando, que no caso será **bigdata_projeto**.<br></br>


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


- **Compilar o programa Java:** Compile o código WordCount.java com o Hadoop.

```
javac -classpath `hadoop classpath` -d . WordCount.java
```

***Explicação do Comando:***

**javac:** Este é o compilador Java, que transforma o código fonte Java (neste caso, *WordCount.java*) em bytecode que pode ser executado pela *Java Virtual Machine (JVM)*.

**-classpath hadoop classpath:** O parâmetro *-classpath* especifica onde o compilador deve procurar as classes necessárias. O comando *hadoop classpath* fornece todos os caminhos de bibliotecas do Hadoop que você precisa para compilar seu código. Assim, ele garante que o código tenha acesso a todas as classes do Hadoop necessárias durante a compilação.

**-d .:** O parâmetro *-d* especifica o diretório de saída onde os arquivos compilados devem ser armazenados. O ponto *.* indica que os arquivos compilados devem ser armazenados no diretório atual.

**WordCount.java:** Este é o arquivo que está sendo compilado.<br></br>


- **Em seguida, crie o arquivo JAR do programa:**

```
jar -cvf wordcount.jar -C . .
```

***Explicação do Comando:***

**jar:** Este é um utilitário para criar arquivos *JAR (Java Archive)*. Um arquivo *JAR* é uma forma de empacotar vários arquivos em um único arquivo, que pode ser executado na *JVM*.

**c:** Esse parâmetro serve para criar um novo arquivo *JAR*.

**v:** Esse parâmetro serve para exibir informações detalhadas durante o processo de criação (verbose).

**f:** Esse parâmetro serve para especificar o nome do arquivo *JAR* que será criado (neste caso, *wordcount.jar*).

**-C . .:** O parâmetro *-C* muda o diretório para o especificado (aqui, *.* significa o diretório atual) e adiciona todos os arquivos encontrados neste diretório ao *JAR*. O segundo ponto *.* é para incluir todos os arquivos e subdiretórios do diretório atual.<br></br>


<br>6. **Executar o programa no Hadoop**

Agora que o programa está pronto, vamos rodá-lo no Hadoop.<br></br>

- **Executar o programa Word Count:** Execute o programa, passando o diretório de entrada ***/input*** e o diretório de saída ***/output***.

```
hadoop jar wordcount.jar WordCount /input /output
```

***Explicação do Comando:***

**hadoop jar:** Este comando inicia a execução de um arquivo *JAR* que contém seu código Java compilado.

**wordcount.jar:** Este é o arquivo *JAR* que você criou, contendo a lógica para contar palavras.

**WordCount:** Este é o nome da classe principal que você definiu no seu código Java.

**/input:** Este é o diretório no *HDFS* onde está o arquivo de entrada (*input.txt*) que você criou.

**/output:** Este é o diretório onde o *Hadoop* irá armazenar os resultados da execução do programa.<br></br>


- **Verificar os resultados:** Depois que o job terminar, visualize a saída com o seguinte comando.
  

```
hdfs dfs -cat /output/part-r-00000
```

Isso exibirá a contagem de cada palavra do arquivo **input.txt**.<br></br>


<br>7. **Objetivo da Aplicação Big Data**

O objetivo dessa aplicação Big Data é demonstrar a capacidade do Hadoop de processar e analisar grandes volumes de dados de forma distribuída. O exemplo da contagem de palavras é um clássico no aprendizado sobre Big Data porque:<br></br>

- **Simplicidade:** É fácil de entender e implementar.

- **Fundamentos do MapReduce:** Introduz os conceitos de Map e Reduce. O Mapper divide a entrada (neste caso, o texto) em palavras e emite pares (palavra, 1). O Reducer pega todos os pares gerados pelo Mapper, agrupa-os pela palavra e soma os valores associados a cada palavra, resultando na contagem total.

- **Compreendimento:** Essa aplicação serve como uma base para compreender como sistemas de Big Data podem lidar com dados em larga escala e realizar análises úteis. 

- **Explorar Novas Possibilidades:** Com esse entendimento, você pode explorar outros casos de uso mais complexos e variados no contexto de Big Data.<br></br>


# Conclusão

Com este tutorial, você aprendeu a configurar o Hadoop no Ubuntu, a carregar dados no HDFS e a criar uma aplicação simples de contagem de palavras usando o modelo MapReduce. 

Esse exemplo pode ser expandido para processar conjuntos de dados maiores, e o mesmo conceito pode ser aplicado a problemas de Big Data mais complexos.

Agora você está pronto para mostrar essa aplicação para outras pessoas e ajudar a disseminar o conhecimento sobre Big Data!

Espero que esse tutorial seja útil para você e para outros interessados em aprender mais sobre Big Data! Se precisar de mais ajustes ou tiver dúvidas, estou à disposição.

# ------------------------------------------------------------------------------



Resultado do Comando hdfs dfs -cat /output/part-r-00000

O comando que você usou para visualizar o resultado:

hdfs dfs -cat /output/part-r-00000
Este comando mostra o conteúdo do arquivo de saída gerado pelo seu programa. O que você obteve é uma contagem de palavras, onde cada linha representa uma palavra do arquivo de entrada e o número correspondente de vezes que essa palavra aparece.

Explicação do Resultado
Aqui está o resultado que você obteve:

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







Você disse que o número à direita de cada palavra indica quantas vezes essa palavra aparece no texto que você processou. Onde está esse texto? Eu posso altera-lo e escrever outro texto no lugar para eu poder executar ele? Posso fazer outros testes?

O ChatGPT disse:
Sim, você pode alterar o texto que está no arquivo input.txt e realizar novos testes! O texto original que você criou estava assim:

Hadoop is an open-source framework
Hadoop is widely used for Big Data
Hadoop processes large datasets


Para Alterar o Texto

Abra o arquivo input.txt para edição usando o vim (ou outro editor de sua preferência):

vim input.txt
Altere o conteúdo do arquivo conforme sua necessidade. Você pode adicionar novas linhas, modificar o texto existente ou até mesmo remover algumas linhas.

Exemplo de novo texto:

Big Data is transforming industries.
Hadoop makes it easy to process large datasets.
Learning Hadoop can be fun and insightful.

Salve e feche o arquivo. No vim, você pode fazer isso pressionando Esc, digitando :wq, e pressionando Enter.


Depois de Alterar o Texto
Após atualizar o arquivo input.txt, você precisa repetir o processo de contagem de palavras para ver os novos resultados:

Coloque o arquivo no HDFS novamente (apenas se você tiver removido o antigo):

hdfs dfs -put input.txt /input

Execute o seu programa de contagem de palavras novamente:

hadoop jar wordcount.jar WordCount /input /output


Verifique o resultado:

hdfs dfs -cat /output/part-r-00000

Observações
Vários Testes: Você pode fazer vários testes, alterando o texto no input.txt, executando o programa e visualizando os resultados quantas vezes quiser.

Limpar Resultados Anteriores: Se você deseja executar testes com diferentes textos, pode ser uma boa ideia excluir os diretórios de saída anteriores. Para fazer isso, você pode usar o seguinte comando antes de executar seu código novamente:

hdfs dfs -rm -r /output

Isso removerá o diretório /output existente, permitindo que você crie um novo diretório de saída ao executar o programa novamente.

Sinta-se à vontade para experimentar! Se tiver mais perguntas ou precisar de ajuda, estou aqui para ajudar!


