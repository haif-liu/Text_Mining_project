# Text_Mining_project
基于Naive Bayes的行业自动分类模型的开发与实现

  **背景：** 为从海量公司招聘信息中提取有用信息为精准推荐做准备，需要对发布招聘信息的各公司所属的行业进行自动分类，使得给定公司文本基本信息实现所属行业的自动分类预测。

  **方法：** 利用爬虫爬取各行业下公司的名称，以公司名称为索引，检索包含各公司基本信息的数据库文件，提取各公司文本数据信息，生成原始训练数据集。经分词处理后，利用卡方检验进行特征的选择，训练Naive Bayes模型进行文本分类。

  **总述：** 针对128个行业，其中一级行业16个，共需分类模型17个，其中1个一级行业分类模型，16个二级行业分类模型。

  ## 1.训练数据的提取与处理

    为学习17个训练模型，需生成17个训练数据集，因此，需要分别提取128个行业下包含公司的文本信息，然后将其分别根据公司所属行业归到相应的数据集中。通过公司名称检索公司数据库，对公司文本数据进行提取，该数据库中包含千万级的公司文本信息，因此，数据提取的第一步是利用爬虫爬取各行业下包含的公司名称，以公司名称为根据，检索公司数据库，得到该行业下的文本数据集。
	
	利用urllib2进行网页的爬取，BeautifulSoup进行网页的解析，按1-128对各行业进行编号，并将爬取和解析到的各行业包含的公司名称保存为*.txt，其中*表示1-128的数字，每个公司占一行。
  
  `关键代码如下：
req = urllib2.Request(//爬取
url='http://www.kanzhun.com/plc'+str(lable_num)+'p'+str(i)+'.html?ka=paging'+str(i),
        headers = headers
            )
 m = urllib2.urlopen(req).read()`
 
	爬取和解析过程中出现的问题及解决方案：
 	(1)爬取过程出现拒绝请求，IP被封的问题。原因在于网站根据流量和日志分析具有反爬虫机制，一旦被识别为爬虫，就会解决该IP的访问。一种解决方法是伪装成浏览器访问，每爬取一个网页，休眠一段时间(50s左右).还有一种方法是使用代理服务器，通过多IP轮询的方式访问，较高效。
 	(2)爬取的网页不能解析的问题。原因不明，解决方法是不对整个网页进行解析，读取包含目标信息的部分，存到新的文件中，对新文件进行解析，解析成功。
 	(3)解析不准确。解决方法是利用正则表达式(import re;)解析,关键代码如下：
  `soup.findAll(name='a',attrs={'ka':re.compile("title$"),'href':True,'target':True,'class':None})`
	
	获取各行业标签数据后，将该行业下的所有公司名称存入list表中，按行读取公司数据库文件，查找该行的第一项(公司名称)是否在list列表中，若在则将该行写入新文件中，直至读取完成，则实现一个行业的所有公司文本数据的抽取。同样的方法抽取其他行业，存入新文件中，命名为*d.txt，其中*表示1-128的数字，即得到各行业的训练集。
	数据抽取过程存在的问题：
	(1)公司数据库中包含缺失样本重复样本，如果重复抽取的话，可能引起样本均衡问题，影响模型的准确率。解决方法是比较各重复样本的长度，优先选择长样本。
	(2)公司数据库中包含缺失样本,缺少对分类有用信息，解决方法是抽取之前检测样本长度，若太短则作丢弃处理。
	将各二级行业训练集归到其所属的一级行业下，归并为新的文件，即得到一级行业训练集，为一级行业的分类做准备。生成训练集后，对各训练集作分词处理，进行特征抽取。
