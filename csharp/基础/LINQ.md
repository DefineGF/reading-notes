#### 实例

##### 确定word出现次数

```c#
private static void GetWordCount(string []words, string term)
{
    var findWord = from word in words
                   where word.ToUpper().Contains(term.ToUpper()) 
                   select word;
    Console.WriteLine($"word {term} occurs {findWord.Count()} times");
}
```

##### 搜索出现次数最多的word

```c#
private static void GetMostCommonWords(string []words)
{
    var frequenctOrder = from word in words
                         where word.Length > 6
                         group word by word into g
                         orderby g.Count() descending
                         select g.Key;
    var commonWords = frequenctOrder.Take(10);
    StringBuilder sb = new StringBuilder();
    sb.AppendLine("The most common words are: ");
    foreach (var v in commonWords)
    {
        sb.AppendLine(" " + v);
    }
    Console.WriteLine(sb.ToString());
}
```



##### 获取最长的单词

```c#
private static string GetLongestWord(string []words)
{
    var longestWord = (from w in words
                       orderby w.Length descending
                       select w).First();
    Console.WriteLine($"The longest word is: {longestWord}");
    return longestWord;
}
```

##### 获取文件夹中.jpg文件

```c#
Task<string[]> task = Task<string[]>.Factory.StartNew(() =>
{
    string path = @"E:\IMAGES";
    string[] files = System.IO.Directory.GetFiles(path);

    var result = (from file in files.AsParallel()
                  let info = new System.IO.FileInfo(file)
                  where info.Extension == ".jpg"
                  select file).ToArray();
    return result;
});
```

