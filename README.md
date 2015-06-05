# Tutorial de introducción a la programación MapReduce en Hadoop
## Introducción
Este tutorial pretende presentar varios ejemplos sencillos que permitan familiarizarse con los conceptos fundamentales del desarrollo de programas en el entorno *MapReduce* de Java, concretamente, en la implementación de Hadoop. Se asume que ya se conocen los aspectos básicos del modelo *MapReduce*. En caso contrario, se recomienda consultar los apuntes de clase y el artículo original que propone este modelo de programación paralela ([MapReduce: Simplified Data Processing on Large Clusters](http://static.googleusercontent.com/media/research.google.com/es//archive/mapreduce-osdi04.pdf) de *Jeffrey Dean* y *Sanjay Ghemawat*), en cuyas ideas se basa la implementación de *MapReduce* de libre distribución incluida en Hadoop.

Este tutorial supone una pequeña introducción al mundo de Hadoop, pero deberías consultar en Internet si deseas disponer de más información.

El tutorial describe como instalar Hadoop, como escribir una primera aplicación, como compilarla, ejecutarla y comprobar la salida.

## Instalación de Hadoop

La forma más fácil de instalar Hadoop es utilizar una de las máquinas virtuales que proporciona Cloudera en su [página web](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cloudera_quickstart_vm.html). Las máquinas virtuales (*Cloudera QuickStart VMs*) traen todo el entorno ya configurado, ahorrando mucho tiempo. Están disponibles para VMWare, KVM y VirtualBox.

El problema fundamental de dichas máquinas virtuales es que requieren bastante memoria RAM (recomendado un mínimo de 4GB dedicados al *guest* según Cloudera). Funcionan con menos memoria, pero el desempeño se reduce (más esperas).

La otra opción es, si disponéis de un sistema operativo GNU/Linux, instalaros Hadoop. Una forma fácil de realizar la instalación es utilizar *Cloudera manager installer*. El [instalador de Cloudera](http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin) debería poder configurar tu equipo de forma fácil.

### Instalación utilizando VirtualBox

1. [Descarga](https://www.virtualbox.org/wiki/Downloads) e instala *VirtualBox* en tu equipo.
2. [Descarga](http://www.cloudera.com/content/cloudera/en/documentation/core/latest/topics/cloudera_quickstart_vm.html) la última versión de la máquina virtual de Cloudera.
3. Descomprime la máquina virtual. Está comprimida con *7zip* (puede que necesites [instalarlo](http://www.7-zip.org/)).
4. Arranca *VirtualBox* y selecciona "Importar servicio virtualizado". Selecciona el archivo OVF ya descomprimido.
5. Una vez terminada la importación (que tardará un rato), debería aparecer la máquina virtual. Vamos a configurar *VirtualBox* para que se cree una red de "solo-anfitrión". `Archivo->Preferencias->Red->Redes solo-anfitrión`. Añádela con los parámetros por defecto. Después, configuramos la máquina virtual para que la use `Botón derecho->Configuración->Red->Adaptador2->Habilitar->Conectado a -> Adaptador solo anfitrión`. De esta forma, podremos acceder por `ssh` a nuestro máquina virtual (utilizando `ssh` o `putty`) a través de la dirección `192.168.56.101`. 
6. Finalmente, arranca la máquina virtual (*paciencia*). Una vez arrancada, deberíamos poder acceder desde el anfitrión a la dirección [http://localhost:8088](http://localhost:8088), dónde podremos ver la interfaz del administrador de recursos. Podrás ver varios que éste y varios puertos está redirigidos por NAT en el Adaptador 1 de tu máquina virtual.
7. El usuario y contraseña por defecto para Cloudera es:
    - User: `cloudera`
    - Password: `cloudera`
8. Puedes utilizar una conexión `sftp://` desde el navegador para facilitar la interacción con Cloudera. Si tu Sistema Operativo anfitrión es GNU/Linux, pulsa `Ctrl+L` y escribe `sftp://cloudera@192.168.56.101/home/cloudera`.

La máquina virtual instada incluye el siguiente *software* (`cloudera-quickstart-vm-5.4.0-0-virtualbox`):

- CentOS 6.4
- JDK (1.7.0_67).
- Hadoop 2.6.0.
- Eclise 4.2.6 (Juno).

## Manejo del HDFS

El sistema de ficheros de Hadoop (HDFS) se puede manejar a través de tres interfaces:

1. Interfaz de línea de comandos, mediante el comando `hadoop fs [opciones]`.
2. Interfaz web (puerto 50070 del *NameNode*). Puedes acceder a través de [http://localhost:50070/](http://localhost:50070/). Ahí podrás ver los ficheros creados y 
3. API de programación.

La interfaz de línea de comandos incluye, por ejemplo, los siguientes comandos:

| Comando | Acción
| --------|---------
| `hadoop fs -ls <path>` | Lista ficheros |
| `hadoop fs -cp <src> <dst>` | Copia ficheros HDFS a HDFS
| `hadoop fs -mv <src> <dst>` | Mueve ficheros HDFS a HDFS |
| `hadoop fs -rm <path>` | Borra ficheros en HDFS |
| `hadoop fs -rmr <path>` | Borra recursivamente |
| `hadoop fs -cat <path>` | Muestra fichero en HDFS |
| `hadoop fs -mkdir <path>` | Crea directorio en HDFS |
| `hadoop fs -chmod ...` | Cambia permisos de fichero |
| `hadoop fs -chown ...` | Cambia propietario/grupo de fichero |
| `hadoop fs -put <local> <dst>` | Copia de local a HDFS |
| `hadoop fs -get <src> <local>` | Copia de HDFS a local |


## El MapReduce nulo

Para entender mejor el modo de operación de MapReduce, comenzamos desarrollando un programa [`Null.java`](code/ejemplo1/Null.java) que, en principio, no hace nada, dejando, por tanto, que se ejecute un trabajo *MapReduce* con todos sus parámetros por defecto:
```java
import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class Null {
	public static void main(String[] args) throws Exception {
		if (args.length != 2) {
  			System.err.println("Uso: null in out");
			System.exit(2);
		}
		// Crea un trabajo MapReduce
		Job job = Job.getInstance(); 
		// Especifica el JAR del mismo
		job.setJarByClass(Null.class);

		// Especifica directorio de entrada y de salida
		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));

		// Arranca el trabajo y espera que termine
		System.exit(job.waitForCompletion(true) ? 0 : 1);
	}
}
```

Lo primero que debes de hacer es crear un directorio, utilizando el CLI de HDFS. Abre una terminal en la máquina virtual (o conéctate por SSH) y escribe:
```bash
    hadoop fs -mkdir input
```
El directorio de trabajo por defecto para este usuario es `\users\cloudera`. Ahora añade algunos ficheros. Para ello, crea con `gedit` los ficheros en local (por ejemplo, `f1.txt` y `f2.txt`), añade el texto que quieras y luego cópialos al HDFS con:
```bash
    hadoop fs -put f*.txt input/
```
Comprueba como ha quedado el sistema de ficheros a través de [http://localhost:50070/](http://localhost:50070/).

Ahora debes crear el fichero [`Null.java`](code/ejemplo1/Null.java) (en local) y compilar y ejecutar este programa especificando como primer parámetro el nombre de ese directorio y como segundo el nombre de un directorio, que no debe existir previamente, donde quedará la salida del trabajo:
```bash
javac  -cp `hadoop classpath` *.java  # compilar
jar cvf Null.jar *.class # crear el JAR
hadoop jar Null.jar Null input output # nombre del JAR, de la clase principal y args del programa
```
Echa un vistazo al contenido del directorio de salida, donde, entre otros, habrá un fichero denominado `part-r-00000`. 
```bash
[cloudera@quickstart ejemplo1]$ hadoop fs -ls output
Found 2 items
-rw-r--r--   1 cloudera cloudera          0 2015-06-05 04:15 output/_SUCCESS
-rw-r--r--   1 cloudera cloudera         77 2015-06-05 04:15 output/part-r-00000
[cloudera@quickstart ejemplo1]$ hadoop fs -ls output/part-r-00000
-rw-r--r--   1 cloudera cloudera         77 2015-06-05 04:15 output/part-r-00000
[cloudera@quickstart ejemplo1]$ hadoop fs -cat output/part-r-00000
0	sadfsadf
0	asdfasfd
9	asdfasdf
9	qwerqwer
18	qwerwqer
18	qwer qwer
27	
28	
```

¿Qué relación ves entre el contenido de este fichero y los ficheros de texto usados en la prueba? Pronto volveremos con ello.

Analicemos mejor el código de [`Null.java`](code/ejemplo1/Null.java). Al especificar un trabajo *MapReduce* tenemos que incluir los siguientes elementos:

> job.setInputFormatClass(TextInputFormat.class);

Esto especifica el formato de entrada. En este caso, hemos usado `TextInputFormat` que es una clase que representa datos de tipo texto y que considera cada línea del fichero como un registro invocando, por tanto, la función **map** del programa por cada línea. Al invocar a **map**, le pasaremos como clave el *offset* (desplazamiento) dentro del fichero correspondiente al principio de la línea. El tipo de la clave será `LongWritable`: `Writable` es el tipo *serializable* que usa *MapReduce* para gestionar todos los datos, que en este caso son de tipo `long`. Como valor, al invocar a **map** pasaremos el contenido de la línea (de tipo `Text`, la versión `Writable` de un `String`).

> job.setMapperClass(Mapper.class);

Esto especifica cuál es la clase utilizada para el **map**. En este caso, utilizamos el Map identidad, que simplemente copia lo que llega a la salida (sin modificarlo).

> job.setMapOutputKeyClass(LongWritable.class);

Este es el tipo de datos de la clave generada por **map**. Dado que la función **map** usada copia la clave recibida, es de tipo `LongWritable`.

> job.setMapOutputValueClass(Text.class);

El tipo de datos del valor generado por **map**. Dado que la función map usada copia el valor recibido, es de tipo `Text`.

> job.setPartitionerClass(HashPartitioner.class);

Esta clase es la que vamos a utilizar para realizar las particiones (decidir que *reduce* se le asigna a cada clave. Por defecto, utilizamos el basado en hash (`hash(key) mod R`).

> job.setNumReduceTasks(1);

Sólo vamos a utilizar un reducer, por eso generamos un solo fichero de salida.

> job.setReducerClass(Reducer.class);

Con esto se especifica la clase del reducer. En este caso, utilizamos el Reducer identidad, que copia los pares `<clave,valor>` que llegan al fichero de salida.

> job.setOutputKeyClass(LongWritable.class);

El tipo de datos de la clave generada por **reduce** y por **map**, excepto si se ha especificado uno distinto para **map** usando `setMapOutputKeyClass`. Dado que la función reduce usada copia la clave recibida, es de tipo `LongWritable`.

> job.setOutputValueClass(Text.class);

El tipo de datos del valor generado por **reduce** y por **map** excepto si se ha especificado uno distinto para **map** usando `setMapValueKeyClass`. Dado que la función reduce usada copia el valor recibido, es de tipo Text.

> job.setOutputFormatClass(TextOutputFormat.class);

Este formato de salida es de tipo texto y consiste en la clave y el valor separados, por defecto, por un tabulador (para pasar a texto los valores generados por reduce, el entorno de ejecución invoca el método `toString` de las respectivas clases `Writable`).

Modifica el código de `Null.java` para especificar dos *reducers* y ejecútalo analizando la salida producida por el programa.

Para terminar esta primera toma de contacto, hay que explicar que el mandato hadoop gestiona sus propios argumentos de la línea de comandos (veremos un ejemplo en la siguiente sección). Es necesario separar dentro de los argumentos de la línea de comandos aquellos que corresponden a Hadoop y los que van destinados a la aplicación. La clase `Tool` facilita este trabajo. A continuación, se presenta la nueva versión de la clase [`Null.java`](code/ejemplo2/Null.java) usando este mecanismo.

```java
import java.io.IOException;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
import org.apache.hadoop.conf.Configured;

public class Null extends Configured implements Tool {
	public int run(String[] args) throws Exception {
		if (args.length != 2) {
  			System.err.println("Usage: null in out");
			System.exit(2);
		}
		Job job = Job.getInstance(getConf()); // le pasa la config.

		job.setJarByClass(getClass()); // pequeño cambio

		FileInputFormat.addInputPath(job, new Path(args[0]));
		FileOutputFormat.setOutputPath(job, new Path(args[1]));
		return job.waitForCompletion(true) ? 0 : 1;
	}

	public static void main(String[] args) throws Exception {
		int resultado = ToolRunner.run(new Null(), args);
		System.exit(resultado);
	}
}
```

## ¡Hola mundo! en Hadoop

El WordCount (contador de palabras) es el "¡Hola Mundo!" de Hadoop. Por su sencillez y su idoneidad para ser resuelto con el paradigma *MapReduce*, se utiliza en multitud de tutoriales de iniciación. Ahora vamos a seguir el [tutorial de iniciación a Hadoop de Cloudera](http://www.cloudera.com/content/cloudera/en/documentation/hadoop-tutorial/CDH5/Hadoop-Tutorial.html) que podemos encontrar en su documentación.

En primer lugar, descarga el código de [WordCount.java](code/ejemplo3/WordCount.java). Cópialo en una carpeta `ejemplo3` del `$HOME` de tu usuario `cloudera`.    

Utiliza un paquete apropiado (y genera la carpeta correspondiente) o mantén el genérico (elimina la línea `package`). Las únicas clases estándar de Java que vamos a utilizar son `IOException` y `regex.Pattern`, que las emplearemos para extraer las palabras de los ficheros:
```java
package master.sd;
import java.io.IOException;
import java.util.regex.Pattern;
```

Esta clase extenderá a la clase `Configured` e implementa la clase de utilidades `Tool`. Haciendo esto, le dices a Hadoop lo que necesita saber para ejecutar tu programa en un objeto de configuración. Luego empleas el `ToolRunner` para ejecutar la aplicación MapReduce:
```java
    import org.apache.hadoop.conf.Configured;
    import org.apache.hadoop.util.Tool;
    import org.apache.hadoop.util.ToolRunner;
```

La clase `Logger` manda mensajes de depuración desde las clases **map** y **reduce**. Cuando ejecutas la aplicación, uno de los mensajes estándar de información proporciona la URL que permite rastrear la ejecución del trabajo. Cualquier mensaje pasado al `Logger` se muestra los logs del map o del reduce de tu servidor Hadoop.
```java
    import org.apache.log4j.Logger;
```

Necesitas las clase `Job` para crear, configurar y ejecutar una instancia de tu aplicación MapReduce. Debes extender la clase `Mapper` utilizando tu propia clase para la acción **map** y añadir las instrucciones específicas de procesado. Lo mismo sucede con el `Reducer`: lo extiendes para crear y personalizar las acciones de tu **reduce**:
```java
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
```

Utiliza la clase `Path` para acceder a tus archivos en el HDFS. En las instrucciones de configuración de tu `Job`, puedes especificar las rutas requeridas utilizando las clases `FileInputFormat` y `FileOutputFormat`:
```java
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
```

Como ya se comentó, los objetos `Writable` tienen métodos para escribir, leer y comparar valores durante el procesamiento de **map** y **reduce**. La clase `Text` es como un `StringWritable`, porque realiza esencialmente las mismas funciones que hacen las clases `IntWritable` para enteros y `LongWritable` para `long`:
```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
```

[WordCount.java](code/ejemplo3/WordCount.java) incluye los métodos `main` y `run` y las clases internas `MiMap` y `MiReduce`.

public class WordCount extends Configured implements Tool {

   private static final Logger LOG = Logger .getLogger( WordCount .class);
The main method invokes ToolRunner, which creates and runs a new instance of WordCount, passing the command line arguments. When the application is finished, it returns an integer value for the status, which is passed to the System object on exit.

public static void main( String[] args) throws  Exception {
     int res  = ToolRunner .run( new WordCount(), args);
     System .exit(res);
  }
The run method configures the job (which includes setting paths passed in at the command line), starts the job, waits for the job to complete, and then returns a Boolean success flag.

public int run( String[] args) throws  Exception {
Create a new instance of the Job object. This example uses the Configured.getConf() method to get the configuration object for this instance of WordCount, and names the job object wordcount.

Job job  = Job .getInstance(getConf(), " wordcount ");
Set the jar to use, based on the class in use.

job.setJarByClass( this .getClass());
Set the input and output paths for your application. You store your input files in HDFS, and then pass the input and output paths as command-line arguments at run time.

FileInputFormat.addInputPaths(job,  new Path(args[ 0]));
     FileOutputFormat.setOutputPath(job,  new Path(args[ 1]));
Set the map class and reduce class for the job. In this case, use the Map and Reduce inner classes defined in this class.

    job.setMapperClass( Map .class);
    job.setReducerClass( Reduce .class);
Use a Text object to output the key (in this case, the word being counted) and the value (in this case, the number of times the word appears).

    job.setOutputKeyClass( Text .class);
    job.setOutputValueClass( IntWritable .class);
Launch the job and wait for it to finish. The method syntax iswaitForCompletion(boolean verbose). When true, the method reports its progress as the Map and Reduce classes run. When false, the method reports progress up to, but not including, the Map and Reduce processes.

In Unix, 0 indicates success, and anything other than 0 indicates a failure. When the job completes successfully, the method returns 0. When it fails, it returns 1.

return job.waitForCompletion( true)  ? 0 : 1;
  }
The Map class (an extension of Mapper) transforms key/value input into intermediate key/value pairs to be sent to the Reducer. The class defines several global variables, starting with an IntWritable for the value 1, and a Text object used to store each word as it is parsed from the input string.

public static class Map extends Mapper<LongWritable ,  Text ,  Text ,  IntWritable > {
    private final static IntWritable one  = new IntWritable( 1);
    private Text word  = new Text();
Create a regular expression pattern you can use to parse each line of input text on word boundaries ("\b"). Word boundaries include spaces, tabs, and punctuation.

private static final Pattern WORD_BOUNDARY = Pattern .compile(" \\ s* \\ b \\ s* ");
Hadoop invokes the map method once for every key/value pair from your input source. This does not necessarily correspond to the intermediate key/value pairs output to the reducer. In this case, the map method receives the offset of the first character in the current line of input as the key, and a Text object representing an entire line of text from the input file as the value. It further parses the words on the line to create the intermediate output.

public void map( LongWritable offset,  Text lineText,  Context context)
        throws  IOException,  InterruptedException {
Convert the Text object to a string. Create the currentWord variable, which you use to capture individual words from each input string.

String line  = lineText.toString();
       Text currentWord  = new Text();
Use the regular expression pattern to split the line into individual words based on word boundaries. If the word object is empty (for example, consists of white space), go to the next parsed object. Otherwise, write a key/value pair to the context object for the job.

for ( String word  : WORD_BOUNDARY .split(line)) {
    if (word.isEmpty()) {
        continue;
   }
       currentWord  = new Text(word);
       context.write(currentWord,one);
   }
}
The mapper creates a key/value pair for each word, composed of the word and the IntWritable value 1. The reducer processes each pair, adding one to the count for the current word in the key/value pair to the overall count of that word from all mappers. It then writes the result for that word to the reducer context object, and moves on to the next. When all of the intermediate key/value pairs are processed, the map/reduce task is complete. The application saves the results to the output location in HDFS.

public static class Reduce extends Reducer<Text ,  IntWritable ,  Text ,  IntWritable > {
     @Override public void reduce( Text word,  Iterable<IntWritable > counts,  Context context)
         throws IOException,  InterruptedException {
       int sum  = 0;
       for ( IntWritable count  : counts) {
        sum  += count.get();
      }
      context.write(word,  new IntWritable(sum));
    }
  }
}
    
## Referencias
Este tutorial se ha realizado basándonos en gran medida en los siguientes tutoriales:

1. [Introducción a la programación MapReduce en Hadoop](http://laurel.datsi.fi.upm.es/docencia/asignaturas/ppd). Universidad Politécnica de Madrid (UPM).

2. [Hadoop Tutorial](http://web.stanford.edu/class/cs246/homeworks/tutorial.pdf) Stanford University.

