---
layout: post
title: FOTA (Firmware Over The Air)
date: 2024-05-06 20:32 +0900
last_modified_at: 2024-05-06 20:32 +0900
tags: [Embedded system]
toc:  true
---
## Conclusion
Embedded System 에서 사용되는 Live Firmware Update System 을 설명한다

## Meaning
- MCU에 물리적으로 접근없이 Server를 연동시켜 자동으로 Fimware Update 이 가능한 시스템.
- **XCP Protocl** 를 이용하여 Version을 체크하여 형상유지에 용이한 Firmware Update를 진행한다.
- Link File을 이용하여 MCU Flash Memory Section을 만들어 Version, 보안체크용 데이터를 지정시켜 보안성에 강화

<!-- ![placeholder](http://placehold.it/800x400 "Large example image") -->

## Concept
IOT이든, 차량이든 어떤 제품에 완제품형태로 결착된 MCU에 직접 물리적으로 접촉하여 펌웨어 업데이트하는건 여간 불편한 일이아니다.

따라서 물리적 연결없이 쉽게 업데이트할수있되, 누구든 쉽게 업데이트할수 없게끔 보안 장치를 걸어둔 부트로더 프로그램을 OTA 프로그램이라고 한다.

차량에서 사용하는 가장 간단하고 기본적인 MCU OTA 흐름도는 다음과 같다.

![placeholder](/fota/fota_1.png "Large example image")


### XCP Protocol?

XCP on CAN , XCP on Ethernet 다양한 물리적 통신에 패킷에 실리는 명령 명세라고 이해하면 편할듯하다.

![placeholder](/fota/fota_2.png/400x200 "Large example image"){: .align-center}


예를들어 Host 에서 보내는 CAN 메세지중 XCP Message Frame 의 PID 에 0x55 , DAQ 에 8000A000 이라는 메세지를 보냈다고 가정하자.

Slave측에서는 0x55는 Flash Memory Read 라는 명령이고 , 읽어야하는 Flash Memory주소는 8000A000 이구나 라고 해석을 하게된다.

이러한 미리 정의된 XCP 명령 프로토콜을 이용하여 Host는 Slave 의 Flash 또는 RAM 에 저장되어있는 값을 받아서 볼수있다.

<mark>이 Flash , RAM 의 저장되어있는 값은 ECU에서 돌고있는 칼만필터의 파라미터일수도, ECU의 ID가 될수도, ECU가 가지고있는 프로그램의 펌웨어 버전이 될수도 있는것</mark>

이러한 XCP Protocol을 이용하여 Flash Write,Erase,Read 명령을 수행하며, 이를 이용하여 ECU와의 Version태그를 비교하며 Live Firmware Update를 수행할수있다.


### MCU (Slave) 측

위 과정에서 사용되는 ECU ID , Version 정보는 모두 MCU측의 FLASH Memory의 특정 Section에 저장되어야한다.

그 이유는 Host 측에서 정확히 ECU 의 어느 주소값에 ECU의 ID, Version 정보가 있는지 알아야 하기때문.


![placeholder](/fota/fota_3.png/400x200 "Large example image"){: .align-center}


다음과같이 Link File을 통해 저장할 공간 (Section)을 만들어준다.

그후 Jenkins와 같은 CI/CD 툴로 Version Tag를 변수로 넣어 자동 컴파일을 걸어두면 쉽게 유지보수가 가능하다.


![placeholder](/fota/fota_4.png/400x200 "Large example image"){: .align-center}


OpenBlt는 다음과같이 Bootloader단과 Application단 Section을 나눈다.

Application 단에서 Firmware Update가 진행되면 Flash Write/Read시 위험할 상황이 생긴다.

또한, 다음과같이 Bootloader단에서 Firmware Update를 진행하면 Update 실패시 재요청하면 되므로 안전하다



### Host -> Slave WorkFlow




Welcome to **Not Pure Poole**! This is an example post to show the layout.
{: .message }

First, do you notice the TOC on the right side? Try to scroll down to read this post, you'll find that the TOC is always sticky in the viewport.

Cum sociis natoque penatibus et magnis <a href="#">dis parturient montes</a>, nascetur ridiculus mus. *Aenean eu leo quam.* Pellentesque ornare sem lacinia quam venenatis vestibulum. Sed posuere consectetur est at lobortis. Cras mattis consectetur purus sit amet fermentum.

> Curabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.

Etiam porta **sem malesuada magna** mollis euismod. Cras mattis consectetur purus sit amet fermentum. Aenean lacinia bibendum nulla sed consectetur.

## Inline HTML elements

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- <mark>To highlight</mark>, use `<mark>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Mark Otto</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

## Footnotes

Footnotes are supported as part of the Markdown syntax. Here's one in action. Clicking this number[^fn-sample_footnote] will lead you to a footnote. The syntax looks like:

{% highlight text %}
Clicking this number[^fn-sample_footnote]
{% endhighlight %}

Each footnote needs the `^fn-` prefix and a unique ID to be referenced for the footnoted content. The syntax for that list looks something like this:

{% highlight text %}
[^fn-sample_footnote]: Handy! Now click the return link to go back.
{% endhighlight %}

You can place the footnoted content wherever you like. Markdown parsers should properly place it at the bottom of the post.

## Heading

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

### Code

Inline code is available with the `<code>` element. Snippets of multiple lines of code are supported through Rouge. Longer lines will automatically scroll horizontally when needed. You may also use code fencing (triple backticks) for rendering code.

{% highlight js %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

You may also optionally show code snippets with line numbers. Add `linenos` to the Rouge tags.

{% highlight js linenos %}
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
{% endhighlight %}

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

### Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

- Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
- Donec id elit non mi porta gravida at eget metus.
- Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

<dl>
  <dt>HyperText Markup Language (HTML)</dt>
  <dd>The language used to describe and define the content of a Web page</dd>

  <dt>Cascading Style Sheets (CSS)</dt>
  <dd>Used to describe the appearance of Web content</dd>

  <dt>JavaScript (JS)</dt>
  <dd>The programming language used to build advanced Web sites and applications</dd>
</dl>

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

### Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](http://placehold.it/800x400 "Large example image")
![placeholder](http://placehold.it/400x200 "Medium example image")
![placeholder](http://placehold.it/200x200 "Small example image")

Align to the center by adding `class="align-center"`:

![placeholder](http://placehold.it/400x200 "Medium example image"){: .align-center}

### Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.

-----

Want to see something else added? <a href="https://github.com/vszhub/not-pure-poole/issues/new">Open an issue.</a>

[^fn-sample_footnote]: Handy! Now click the return link to go back.
