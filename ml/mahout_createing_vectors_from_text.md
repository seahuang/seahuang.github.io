
# Mahout如何文本转向量

## 介绍
> 为了将一批文件进行聚类和分类操作，通常需要将原始的文本文件转换成向量，只有向量才能被聚类算法使用。下文描述了这种方法。

## 来自Lucene

## 来自一个目录下的文本文件
> Mahout有一些工具类可以将一个目录下的文本文件转换成向量。在生成向量前，需要将这些文本文件转换成序列文件（SequenceFile）格式。序列文件（SequenceFile）是一个hadoop类，这种序列文件允许我们写入任意key-value对。DocumentVectorizer要求key必须是Text类型，key是文档唯一的ID，值（value）必须是UTF-8格式的Text类型。

[Tika](http://tika.apache.org/)可以帮助将二进制文件转变成文本文件。

### 目录下的文档转成序列文件（SequenceFile）格式
> Mahout有一个漂亮的工具类，它可以读取一个目录（包括目录下的子目录）下的文档，并生成序列文件（SequenceFile），序列文件（SequenceFile）是采用分块方式存放。

```
$MAHOUT_HOME/bin/mahout seqdirectory 
    --input (-i) input                       Path to job input directory.   
    --output (-o) output                     The directory pathname for     
                                                 output.                        
    --overwrite (-ow)                        If present, overwrite the      
                                                 output directory before        
                                                 running job                    
    --method (-xm) method                    The execution method to use:   
                                                 sequential or mapreduce.       
                                                 Default is mapreduce           
    --chunkSize (-chunk) chunkSize           The chunkSize in MegaBytes.    
                                                 Defaults to 64                 
    --fileFilterClass (-filter) fFilterClass The name of the class to use   
                                                 for file parsing. Default:     
                                                 org.apache.mahout.text.PrefixAdditionFilter                   
    --keyPrefix (-prefix) keyPrefix          The prefix to be prepended to  
                                                 the key                        
    --charset (-c) charset                   The name of the character      
                                                 encoding of the input files.   
                                                 Default to UTF-8 {accepts: cp1252|ascii...}             
    --method (-xm) method                    The execution method to use:   
                                                 sequential or mapreduce.       
                                             Default is mapreduce           
    --overwrite (-ow)                        If present, overwrite the      
                                                 output directory before        
                                                 running job                    
    --help (-h)                              Print out help                 
    --tempDir tempDir                        Intermediate output directory  
    --startPhase startPhase                  First phase to run             
    --endPhase endPhase                      Last phase to run  
```
