# SiNiHu

SiNiHu is an AI-powered visual creation studio. Users make presentations and other visual content with AI providers they already pay for.

## Language

**SiNiHu**:
The product name.
_Avoid_: Slider, OpenDeck

**BYOK (Bring Your Own Key)**:
The principle that the user owns the AI access SiNiHu uses. That can be their API keys or their existing AI subscription or SDK login. SiNiHu never resells AI usage.
_Avoid_: AI credits, bundled AI

**Card**:
The single core unit of content in a presentation, the thing a user edits, regenerates and presents one at a time.
_Avoid_: Slide, page

**Frame**:
The outer shape of a Card: either a fixed aspect ratio (such as 16:9) or fluid (height follows content).
_Avoid_: Canvas, card size

**Layout**:
A named arrangement of Regions that a Card uses, such as "title + two columns". Switching a Card's Layout re-flows its content.
_Avoid_: Template (a template is a whole starting deck), smart layout

**Region**:
A named area of a Layout. Its Elements flow in order inside it.
_Avoid_: Slot, placeholder, zone

**Element**:
A single piece of content on a Card: text, image, shape, table, chart and so on.
_Avoid_: Block, component, object

**Compound Element**:
An Element made of repeated structured items, such as an item grid of icon + heading + text. It carries structure that would otherwise need nested Regions.
_Avoid_: Smart layout, nested layout

**Free Element**:
An Element the user has pulled out of its Region and placed at an absolute position on the Card. It remembers its home Region and can snap back.
_Avoid_: Freeform box, floating element
