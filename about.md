---
layout: page
title: About
permalink: /about/
res: /assets/about
brands: ["brand-asuper=https://www.australiansuper.com/", 'brand-carsguide=http://www.carsguide.com.au/', 'brand-commbank=https://www.commbank.com.au/', 'brand-corelogic=http://www.corelogic.com.au/', "brand-nhds=http://www.homedoctor.com.au/", 'brand-paypal=https://www.paypal.com/', 'brand-quantas=http://www.qantas.com.au/', 'brand-tedx=http://tedxsydney.com/', 'brand-telstra=https://www.telstra.com.au/']
---

Hey my name is Coby and I'm an Android developer at [Vivant][Vivant]. Working there I've been lucky enough to work with some truly fantastic brands including:

&nbsp;  

<div class="mdl-grid about-grid">
	{% for brand in page.brands %}

	<div class="mdl-cell mdl-cell--2-col mdl-shadow--4dp">
		{% assign componenets = brand | split:"=" %}
		<a class="mdl-button mdl-js-button mdl-js-ripple-effect about-button" href="{{ componenets[1] }}">
			<img src="{{ page.res }}/{{ componenets[0]}}.png" />
		</a>
	</div>

	{% endfor %}
</div>

&nbsp;

I love learning new and funky things and want to share everything I can with anyone I can. Hopefully in my time I've come across something you will find useful!

&nbsp;

I'm no web developer but if you want to see how this site is built [you are welcome to view the source here][Github Blog]


[Vivant]: http://vivant.com.au/
[Github Blog]: https://github.com/cplain/cplain.github.io