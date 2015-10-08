---
layout: page
title: About
permalink: /about/
res: /assets/about
brands: ['brand-asuper', 'brand-carsguide', 'brand-commbank', 'brand-corelogic', 'brand-paypal', 'brand-quantas', 'brand-tedx', 'brand-telstra']
brand_urls: ['https://www.australiansuper.com/', 'http://www.carsguide.com.au/', 'https://www.commbank.com.au/', 'http://www.corelogic.com.au/', 'https://www.paypal.com/', 'http://www.qantas.com.au/', 'http://tedxsydney.com/', 'https://www.telstra.com.au/']
---

Hey my name is Coby and I'm an Android developer at [Vivant][Vivant]. Working there I've been lucky enough to work with some truly fantastic brands including:

&nbsp;  

<div class="mdl-grid about-grid">
	{% for brand in page.brands %}

	<div class="mdl-cell mdl-cell--2-col mdl-shadow--4dp">
		<a class="mdl-button mdl-js-button mdl-js-ripple-effect about-button" href="{{ page.brand_urls[forloop.index0] }}">
			<img src="{{ page.res }}/{{ brand }}.png" />
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