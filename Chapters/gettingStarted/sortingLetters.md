## Little widget with Layout

### Sorting letters?

![event propagation](figures/letters_drag_drop.png)

### Drag and drop

#### Element containers

drag >> lettersSorterRoundedContainerWithBorder

```smalltalk
^ self lettersSorterContainer
    border: (BlBorderBuilder new paint: Color gray; dashed; width: 2; build);
    geometry: (BlRoundedRectangleGeometry cornerRadius: 5)
```

drag >> lettersSorterContainer

```smalltalk
^ BlElement new
        layout: BlFlowLayout horizontal;
        constraintsDo: [ :c | c horizontal matchParent. c vertical matchParent ];
        padding: (BlInsets all: 10)
```

drag >> labelContainer: anElement with: aText

```smalltalk
^ BlElement new
        layout: (BlLinearLayout vertical alignTopCenter cellSpacing: 10);
        constraintsDo: [ :c | c horizontal matchParent. c vertical matchParent ];
        addChild: (BlTextElement new text: aText asRopedText);
        addChild: anElement
```

drag >> letterFor: aCharacter

```smalltalk
^ BlElement new
        layout: BlLinearLayout horizontal alignCenter;
        size: 30 @ 30;
        margin: (BlInsets all: 5);
        background: Color veryVeryLightGray;
        border: (BlBorder paint: Color veryLightGray width: 1);
        geometry: (BlRoundedRectangleGeometry cornerRadius: 3); width: 2 offset: 0 @ 0);
        addChild: (BlTextElement new labelMeasurement; text: aCharacter asString asRopedText)
```

#### drag containers event

lettersSorterDraggableLetterFor: aCharacter

```smalltalk
| element offset |
element := self letterFor: aCharacter.
element addEventHandlerOn: BlDragStartEvent do: [ :event |
    event consumed: true.
    self inform: 'source1 BlStartDragEvent'.
    offset := event position - element position.
    element removeFromParent ].

"element addEventHandlerOn: BlDragEndEvent do: [ :event |
    event consumed: true.
    self inform: 'source1 BlDragEndEvent' ]."

element addEventHandlerOn: BlDragEvent do: [ :event |
    event consumed: true. "self inform:  'source1 BlDragEvent'."
    element position: event position - offset.
    element hasParent ifFalse: [
        BlParallelUniverse all first spaces first root addChild: element ].
    element preventMeAndChildrenMouseEvents ].

^ element
```

#### drop containers event

drag >> vowelsDropContainer

```smalltalk
| vowels |
vowels := self lettersSorterRoundedContainerWithBorder.
vowels background: Color lightRed.
vowels addEventHandlerOn: BlDropEvent do: [ :event | event consumed: true. self inform: 'target BlDropEvent'.
        (event gestureSource firstChild text first isCharacter and: [ event gestureSource firstChild text first isVowel ])
        ifTrue: [  self inform: 'drop accepted'. event gestureSource removeFromParent. event target addChild: event gestureSource  ]
        ifFalse: [  self inform: 'drop rejected'. event gestureSource position: 100 @ 400; allowMeAndChildrenMouseEvents] ].

vowels addEventHandlerOn: BlDragEnterEvent do: [ :event | event consumed: true. self inform: 'target BlDragEnterEvent' ].
vowels addEventHandlerOn: BlDragLeaveEvent do: [ :event | event consumed: true. self inform: 'target BlDragLeaveEvent' ].

^ vowels
```

drag >>  consonantsDropContainer

```smalltalk
| consonants |
consonants := self lettersSorterRoundedContainerWithBorder.
consonants background: Color lightBlue.
consonants addEventHandlerOn: BlDropEvent do: [ :event | event consumed: true. self inform: 'target BlDropEvent'.
        (event gestureSource firstChild text first isCharacter and: [ event gestureSource firstChild text first isVowel not ])
        ifTrue: [  self inform: 'drop accepted'. event gestureSource removeFromParent. event target addChild: event gestureSource  ]
        ifFalse: [  self inform: 'drop rejected'. event gestureSource position: 100 @ 400; allowMeAndChildrenMouseEvents] ].

^ consonants
```

####  entry point

drag >> lettersSorter

```smalltalk
<sampleInstance>
| letters vowels consonants |
letters := self lettersSorterContainer.
vowels := self vowelsDropContainer.
consonants := self consonantsDropContainer.

letters addChildren: { $a. $c. $Q. $o. $j. $E. $y. $Z. $U. $B. $p. $i } 
    collect: [ :each | self lettersSorterDraggableLetterFor: each ]).

^ BlElement new
        layout: (BlLinearLayout horizontal cellSpacing: 30);
        constraintsDo: [ :c | c horizontal matchParent. c vertical matchParent ];
        addChildren: {
                (self labelContainer: letters with: 'Letters to sort').
                (self labelContainer: vowels with: 'Vowel letters').
                (self labelContainer: consonants with: 'Consonant letters') };
        openInNewSpace
```