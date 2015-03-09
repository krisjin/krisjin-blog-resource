title: TF-IDF关键字提取
date: 2015-03-09 09:32:47
categories: NLP
tags: [NLP,TF-IDF,关键字]
---

TF-IDF（term frequency–inverse document frequency）是一种用于资讯检索与文本挖掘的常用加权技术。TF-IDF是一种统计方法，用以评估一字词对于一个文件集或一个语料库中的其中一份文件的重要程度。字词的重要性随着它在文件中出现的次数成正比增加，但同时会随着它在语料库中出现的频率成反比下降。TF-IDF加权的各种形式常被搜索引擎应用，作为文件与用户查询之间相关程度的度量或评级。除了TF-IDF以外，互联网上的搜索引擎还会使用基于连结分析的评级方法，以确定文件在搜寻结果中出现的顺序。 <!--more-->

**想要了解更多请查看**：   
[http://www.ruanyifeng.com/blog/2013/03/tf-idf.html](http://www.ruanyifeng.com/blog/2013/03/tf-idf.html "TF-IDF与余弦相似性的应用（一）：自动提取关键词")

[http://zh.wikipedia.org/wiki/TF-IDF](http://zh.wikipedia.org/wiki/TF-IDF "TF-IDF")


实现代码清单：

	package net.snails.common.algorithm.keyword;

	import java.util.ArrayList;
	import java.util.Collections;
	import java.util.Comparator;
	import java.util.LinkedList;
	import java.util.List;
	import java.util.Map;
	import java.util.Map.Entry;
	import java.util.TreeMap;
	import java.util.concurrent.atomic.AtomicInteger;
	
	import net.snails.common.algorithm.summary.StopWord;
	import net.snails.common.algorithm.util.CorpusLoad;
	
	import org.ansj.domain.Term;
	import org.ansj.splitWord.analysis.ToAnalysis;
	
	import com.google.common.base.Strings;
	
	/**
	 * TF-IDF关键词提取
	 * 
	 * @author krisjin
	 * 
	 */
	public class TFIDF {

	/**
	 * 语料文档总数
	 */
	private int DOC_NUM;

	/**
	 * 文档中划分的句子的词频统计
	 */
	private Map<String, Integer>[] FRAGMENT_TF;

	/**
	 * 词频
	 */
	private Map<String, Integer> TF;

	/**
	 * 逆文档词频
	 */
	private Map<String, Double> IDF;

	/**
	 * 将文档 进行片段式的划分
	 * 
	 * @param doc
	 * @return
	 */
	private List<String> getFragmentList(String doc) {
		List<String> sentences = new ArrayList<String>();

		if (doc == null)
			return sentences;

		String[] senArr = doc.split("[，,。:：“”？?！!；;]");

		for (String sen : senArr) {
			sen = sen.trim();
			if (Strings.isNullOrEmpty(sen))
				continue;
			sentences.add(sen);
		}

		return sentences;
	}

	private void process(List<List<String>> wordFragment) {
		FRAGMENT_TF = new Map[wordFragment.size()];
		TF = new TreeMap<String, Integer>();
		IDF = new TreeMap<String, Double>();
		int index = 0;

		for (List<String> frag : wordFragment) {
			Map<String, Integer> tmpTF = new TreeMap<String, Integer>();
			for (String word : frag) {
				Integer freq = tmpTF.get(word);
				freq = (freq == null ? 0 : freq) + 1;
				tmpTF.put(word, freq);
			}
			FRAGMENT_TF[index] = tmpTF;

			for (Map.Entry<String, Integer> entry : tmpTF.entrySet()) {
				String word = entry.getKey();
				Integer freq = TF.get(word);

				freq = (freq == null ? 0 : freq) + 1;
				TF.put(word, freq);

			}
			++index;
		}

		for (Map.Entry<String, Integer> entry : TF.entrySet()) {
			String word = entry.getKey();

			IDF.put(word, Math.log(DOC_NUM / (getIncludeWordDocs(FRAGMENT_TF, word) + 1)));

		}

	}

	private int getIncludeWordDocs(Map<String, Integer>[] fragmentDocs, String word) {
		AtomicInteger ai = new AtomicInteger();
		for (Map<String, Integer> frag : fragmentDocs) {

			if (frag.containsKey(word)) {
				ai.incrementAndGet();
			}
		}
		return ai.get();
	}

	private List<Map.Entry<String, Double>> getKeywordList() {

		Map<String, Double> result = new TreeMap<String, Double>();

		for (Map.Entry<String, Integer> entry : TF.entrySet()) {

			String word = entry.getKey();
			Integer freq = entry.getValue();

			if (IDF.containsKey(word)) {
				double val = ((double) freq / (double) TF.size()) * (IDF.get(word));
				result.put(word, val);
			} else {
				result.put(word, ((double) freq / TF.size()));
			}

		}

		List<Map.Entry<String, Double>> list = new ArrayList<Map.Entry<String, Double>>(result.entrySet());

		Collections.sort(list, new MapValueComparator());
		return list;
	}

	private List<List<String>> getParticipleFragment(List<String> sentences) {

		List<List<String>> wordFragment = new ArrayList<List<String>>();

		for (String sens : sentences) {
			List<String> wordList = new LinkedList<String>();
			List<Term> termList = ToAnalysis.parse(sens);

			for (Term term : termList) {
				if (validateWord(term)) {
					if (!(term.item().name == null)) {
						wordList.add(term.getName());
					}
				}
			}

			if (wordList.size() > 0)
				wordFragment.add(wordList);
		}

		DOC_NUM = wordFragment.size();

		return wordFragment;
	}

	public List<String> extractKeyword(String doc, int size) {

		List<String> result = new ArrayList<String>();

		List<List<String>> fragList = getParticipleFragment(getFragmentList(doc));

		process(fragList);

		List<Map.Entry<String, Double>> keywords = getKeywordList();

		if (keywords.size() <= size) {
			for (Map.Entry<String, Double> entry : keywords) {
				result.add(entry.getKey());
			}
		} else {
			int i = 1;
			for (Map.Entry<String, Double> entry : keywords) {
				result.add(entry.getKey());
				if (i == size)
					break;
				i++;
			}
		}

		return result;
	}

	public static boolean validateWord(Term term) {
		if (term.getNatureStr().startsWith("n") || term.getNatureStr().startsWith("v") || term.getNatureStr().startsWith("d")
				|| term.getNatureStr().startsWith("a")) {
			if (!StopWord.contains(term.getName())) {
				return true;
			}
		}
		return false;
	}

	}

	class MapValueComparator implements Comparator<Map.Entry<String, Double>> {

	public int compare(Entry<String, Double> o1, Entry<String, Double> o2) {
		return o2.getValue().compareTo(o1.getValue());
	}

	public static void main(String[] args) {

		TFIDF ti = new TFIDF();

		String corpus = CorpusLoad.getText("extract.txt");

		List<String> kds = ti.extractKeyword(corpus, 12);

		for (String k : kds) {
			System.out.println(k);
		}

	}
	}


上述代码简单实现了TF-IDF算法，欢迎讨论！

github 查看：[https://github.com/zoopaper/common-algorithm.git](https://github.com/zoopaper/common-algorithm.git)