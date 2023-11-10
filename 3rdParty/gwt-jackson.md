> # gwt-jackson


* repo：https://github.com/nmorel/gwt-jackson
* jackson-annotation：https://github.com/FasterXML/jackson-annotations


Immutable class
===============

```Java
public class Card {
	public final Suit suit;
	public final int number;

	@JsonCreator
	public Card(@JsonProperty("suit") Suit suit, @JsonProperty("number") int number) {
		this.suit = suit;
		this.number = number;
	}
}
```


沒有 _etter 的 class
====================

```Java
@JsonAutoDetect(fieldVisibility=JsonAutoDetect.Visibility.ANY)
public class Deck {
	private List<Card> sequence = Lists.newArrayList();
}
```
