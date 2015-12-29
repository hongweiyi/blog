title: Python - 遍历目录
tags:
  - Python
id: 228
categories:
  - 技术分享
date: 2011-12-21 12:21:13
---

Python遍历目录经常要用到，所以就在日志mark一下吧。
 <!--more-->  

最开始是用传统的方式写的，找到目录，list它的所有下一级目录，再递归遍历，代码如下：
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">import</span><span>&#160;</span><span class="commonlibs">os</span><span>, </span><span class="commonlibs">sys</span><span>&#160; </span></span>
2.  <span>&#160; </span>
3.  <span></span><span class="comment"># list files in root by recursion </span><span>&#160; </span></span>
4.  <span></span><span class="keyword">def</span><span> listfile(root, callback = None):&#160;&#160; </span></span>
5.  <span>&#160;&#160;&#160; </span><span class="keyword">for</span><span> path </span><span class="keyword">in</span><span>&#160;</span><span class="commonlibs">os</span><span>.listdir(root):&#160;&#160; </span></span>
6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; abs_path = </span><span class="commonlibs">os</span><span>.path.join(root, path)&#160;&#160; </span></span>
7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span>&#160;</span><span class="commonlibs">os</span><span>.path.isdir(abs_path):&#160;&#160; </span></span>
8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; listfile(abs_path, callback)&#160;&#160; </span>
9.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">else</span><span>:&#160;&#160; </span></span>
10.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> callback: callback(abs_path)&#160;&#160; </span></span>
11.  <span>&#160; </span>
12.  <span></span><span class="comment">#do something to the path </span><span>&#160; </span></span>
13.  <span></span><span class="keyword">def</span><span> dosth(path):&#160;&#160; </span></span>
14.  <span>&#160;&#160;&#160; </span><span class="keyword">print</span><span> path&#160;&#160; </span></span>
15.  <span>&#160; </span>
16.  <span></span><span class="keyword">if</span><span>&#160;</span><span class="builtins">__name__</span><span> == &quot;</span><span class="builtins">__main__</span><span>&quot;:&#160;&#160; </span></span>
17.  <span>&#160;&#160;&#160; path = </span><span class="builtins">raw_input</span><span>('enter your path: ')&#160;&#160; </span></span>
18.  <span>&#160;&#160;&#160; </span><span class="keyword">try</span><span>:&#160;&#160; </span></span>
19.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; listfile(path, dosth)&#160;&#160; </span>
20.  <span>&#160;&#160;&#160; </span><span class="keyword">except</span><span>: </span><span class="comment"># Exception in permissions, sytem file etc </span><span>&#160; </span></span>
21.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">pass</span><span>&#160; </span></span>         </div>       </td>     </tr>   </tbody></table>  

看了上面的代码总觉得别扭，连轮子都有的python应该有遍历目录的方法吧，Google之后发现真有，即os.walk(path)方法。

os.walk()可以得到一个三元元组(dirpath, dirnames, filenames)，dirpath是字符串，为起始路径；dirnames是list，为dirpath下的文件夹名；filenames是list，为dirpath下的文件名。

其中dirnames以及filenames均不包含路径信息，如果需要得到全路径，需要使用os.path.join(path,name)。
  <table border="1" cellspacing="0" cellpadding="0"><tbody>     <tr>       <td valign="top" width="568">         <div class="dp-highlighter">           <div class="bar"></div>            

1.  <span><span class="keyword">import</span><span>&#160;</span><span class="commonlibs">os</span><span>, </span><span class="commonlibs">sys</span><span>&#160; </span></span>2.  <span>&#160; </span>3.  <span></span><span class="comment"># list files in root by os.walk() </span><span>&#160; </span></span>4.  <span></span><span class="keyword">def</span><span> walkfile(root, callback = None):&#160;&#160; </span></span>5.  <span>&#160;&#160;&#160; </span><span class="keyword">for</span><span> dirpath, dirnames, filenames </span><span class="keyword">in</span><span>&#160;</span><span class="commonlibs">os</span><span>.walk(root):&#160;&#160; </span></span>6.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">for</span><span> filename </span><span class="keyword">in</span><span> filenames:&#160;&#160; </span></span>7.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; abs_path = </span><span class="commonlibs">os</span><span>.path.join(dirpath, filename)&#160;&#160; </span></span>8.  <span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; </span><span class="keyword">if</span><span> callback: callback(abs_path)&#160;&#160; </span></span>9.  <span>&#160; </span>10.  <span></span><span class="comment">#do something to the path </span><span>&#160; </span></span>11.  <span></span><span class="keyword">def</span><span> dosth(path):&#160;&#160; </span></span>12.  <span>&#160;&#160;&#160; </span><span class="keyword">print</span><span> path&#160;&#160; </span></span>13.  <span>&#160; </span>14.  <span></span><span class="keyword">if</span><span>&#160;</span><span class="builtins">__name__</span><span> == &quot;</span><span class="builtins">__main__</span><span>&quot;:&#160;&#160; </span></span>15.  <span>&#160;&#160;&#160; path = </span><span class="builtins">raw_input</span><span>('enter your path: ')&#160;&#160; </span></span>16.  <span>&#160;&#160;&#160; </span><span class="comment">#In theory, there is no permissions/systemfile exception by os.walk </span><span>&#160; </span></span>17.  <span>&#160;&#160;&#160; walkfile(path, dosth)&#160;&#160; </span>         </div>       </td>     </tr>   </tbody></table>
