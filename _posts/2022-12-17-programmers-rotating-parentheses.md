---
layout: post
title: "알고리즘 풀이 - 괄호 회전하기"
date: 2022-12-17
category: Algorithm
tags: [algorithm, stack, programmers, datastructures]
---
# 알고리즘 풀이 - 괄호 회전하기
[문제출처: 프로그래머스](https://school.programmers.co.kr/learn/courses/30/lessons/76502)

# 풀이 방법
- 분할과 정복을 활용하여 풀 수 있다.
- 문자 배열 회전의 각 시작지점이 첫 인덱스가 되도록 하는 새로운 리스트로 반환해주는 메서드를 작성한다.(`arrange`)
- 인덱스 i 부터 i-1 까지 순회하기 위한 메서드를 작성한다(`getIndex`)
- 재배열된 리스트로 괄호가 올바르게 닫혔는지 검사하는 메서드를 작성한다(`correct`)
- 괄호가 올바르게 닫혔는지 검사하는 메서드를 작성한다(`isOpenClosed`)

# 테스트 코드
```java
class RotatingParenthesesTest {

	RotatingParentheses r = new RotatingParentheses();

	@Test
	void solution() {
		assertThat(r.solution("[](){}")).isEqualTo(3);
		assertThat(r.solution("}]()[{")).isEqualTo(2);
		assertThat(r.solution("[)(]")).isEqualTo(0);
		assertThat(r.solution("}}}")).isEqualTo(0);
	}

	@Test
	void isOpenClosed() {
		assertThat(r.isOpen("[")).isTrue();
    assertThat(r.isOpen("]")).isFalse();
	}

	@Test
	void isCorrect() {
		assertThat(r.isCorrect(0, "[](){}")).isTrue();
	}

	@Test
	void getIndex() {
		assertThat(r.getIndex(0, 0, 5)).isEqualTo(0);
		assertThat(r.getIndex(0, 4, 5)).isEqualTo(4);
		assertThat(r.getIndex(4, 0, 5)).isEqualTo(4);
		assertThat(r.getIndex(4, 1, 5)).isEqualTo(0);
		assertThat(r.getIndex(4, 2, 5)).isEqualTo(1);
		assertThat(r.getIndex(4, 3, 5)).isEqualTo(2);
	}

	/**
	 * 0	"[](){}"	O
	 * 1	"](){}["	X
	 * 2	"(){}[]"	O
	 * 3	"){}[]("	X
	 * 4	"{}[]()"	O
	 * 5	"}[](){"	X
	 */
	@Test
	void arrange() {
		assertThat(r.arrange(0, "[](){}".toCharArray()))
			.containsExactlyElementsOf(toCharList("[](){}"));
		assertThat(r.arrange(1, "[](){}".toCharArray()))
			.containsExactlyElementsOf(toCharList("](){}["));
		assertThat(r.arrange(2, "[](){}".toCharArray()))
			.containsExactlyElementsOf(toCharList("(){}[]"));
		assertThat(r.arrange(3, "[](){}".toCharArray()))
			.containsExactlyElementsOf(toCharList("){}[]("));
		assertThat(r.arrange(4, "[](){}".toCharArray()))
			.containsExactlyElementsOf(toCharList("{}[]()"));
		assertThat(r.arrange(5, "[](){}".toCharArray()))
			.containsExactlyElementsOf(toCharList("}[](){"));
	}

	private List<Character> toCharList(String s) {
		return s.chars().mapToObj(c -> (char) c).collect(Collectors.toList());
	}
	@Test
	void isCorrect2() {
		assertThat(r.isCorrect(toCharList("[](){}"))).isTrue();
		assertThat(r.isCorrect(toCharList("](){}["))).isFalse();
		assertThat(r.isCorrect(toCharList("(){}[]"))).isTrue();
		assertThat(r.isCorrect(toCharList("){}[]("))).isFalse();
		assertThat(r.isCorrect(toCharList("{}[]()"))).isTrue();
		assertThat(r.isCorrect(toCharList("}[](){"))).isFalse();
	}
}
```

# 풀이 코드
```java
import java.util.ArrayDeque;
import java.util.ArrayList;
import java.util.Deque;
import java.util.List;

public class RotatingParentheses {

	char[] opens = new char[] {'[', '{', '('};
	char[] closes = new char[] {']', '}', ')'};

	public int solution(String s) {
		int N = s.length();
		int totalCorrect = 0;
		for (int i = 0; i < N; i++) {
			totalCorrect += isCorrect(i, s) ? 1 : 0;
		}
		return totalCorrect;
	}

	public boolean isCorrect(int startIndex, String s) {
		return isCorrect(startIndex, s.toCharArray());
	}

	public boolean isCorrect(int startIndex, char[] parenthesesChar) {
		List<Character> parentheses = arrange(startIndex, parenthesesChar);
		return isCorrect(parentheses);
	}

	public boolean isCorrect(List<Character> parentheses) {
		Deque<Character> deque = new ArrayDeque<>();
		for (Character paren : parentheses) {
			if (deque.isEmpty()) {
				deque.add(paren);
				continue;
			}
			if (isOpen(paren)) {
				deque.add(paren);
				continue;
			}
			Character prevParen = deque.pollLast();
			if (!isOpenClose(prevParen, paren)) {
				return false;
			}
		}
		// check remaining
		while (!deque.isEmpty()) {
			Character open = deque.pollFirst();
			if (deque.isEmpty()) {
				return false;
			}
			Character close = deque.pollFirst();
			if (!isOpenClose(open, close)) {
				return false;
			}
		}

		return true;
	}

	private boolean isOpenClose(Character open, Character close) {
		return (isOpen(open) && isClose(close))&& findOpens(open) == findCloses(close);
	}

	private int findCloses(Character close) {
		return findIndex(close, closes);
	}

	private int findOpens(Character open) {
		return findIndex(open, opens);
	}

	private int findIndex(Character findChar, char[] candidates) {
		for (int i = 0; i < candidates.length; i++) {
			if (candidates[i] == findChar) {
				return i;
			}
		}
		throw new IllegalArgumentException();
	}

	public List<Character> arrange(int startIndex, char[] parentheses) {
		int L = parentheses.length;
		List<Character> deque = new ArrayList<>();
		for (int i = 0; i < L; i++) {
			int index = getIndex(startIndex, i, L);
			char p = parentheses[index];
			deque.add(p);
		}
		return deque;
	}

	public int getIndex(int startIndex, int i, int length) {
		return (i+startIndex) % length;
	}

	public boolean isOpen(String s) {
		return isOpen(s.toCharArray()[0]);
	}

	private boolean isClose(char other) {
		return !isOpen(other);
	}

	private boolean isOpen(char other) {
		for (char c : opens) {
			if (c == other) {
				return true;
			}
		}
		return false;
	}
}
```
