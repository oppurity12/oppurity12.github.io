---
permalink: /
title: "Jeongyeon Hwang"
author_profile: false
redirect_from: 
  - /about/
  - /about.html
---

<style>
/* Hide auto-rendered page title since we render it inside the profile header */
.page__title { display: none; }

.page a {
  color: var(--global-text-color);
  text-decoration: underline;
}
.page a:hover {
  color: var(--global-link-color);
  text-decoration: underline;
}

.page__content {
  line-height: 1.7;
  color: var(--global-text-color);
}

.page__content p {
  margin-bottom: 1.1em;
}

.selected-pubs .archive__item {
  margin-bottom: 1.2em;
}
</style>

<div class="profile-header">
  <img class="profile-header__photo" src="{{ '/images/profile.jpg' | relative_url }}" alt="Jeongyeon Hwang" />
  <div class="profile-header__info">
    <h1>Jeongyeon Hwang</h1>
  </div>
</div>

Hi! I’m Jeongyeon Hwang, a fourth year integrated M.S./Ph.D. candidate in the Graduate School of Artificial Intelligence at POSTECH, advised by [Jungseul Ok](https://sites.google.com/view/jungseulok).

My research focuses on improving the reliability of machine learning and natural language processing systems in real-world settings. In particular, I study large language models (LLMs), with an emphasis on robustness against adversarial inputs, corrupted training data, and potential misuse such as fake content generation. Recently, I have been especially interested in watermarking techniques for detecting LLM-generated text.

 
Please feel free to reach out via email at *{firstname}.{lastname}@postech.ac.kr*.

## News

<ul class="news-list">
<li><span class="news-date">May 2026</span>A paper (<a href="https://arxiv.org/abs/2509.23019" target="_blank">BIRA</a>) got accepted at ICML 2026.</li>
<li><span class="news-date">Jan 2026</span>Received an <span style="color: #0e7490;">Honorable Mention</span> at the BK21 Outstanding Paper Awards from POSTECH GSAI.</li>
<li><span class="news-date">Sep 2025</span>Beginning a research visit at <a href="https://www.nyu.edu/" target="_blank">NYU</a> (September–December).</li>
<li><span class="news-date">Aug 2025</span>Two papers (<a href="https://arxiv.org/abs/2410.22954" target="_blank">RA-RAG</a>, <a href="https://arxiv.org/abs/2506.00344" target="_blank">LSC</a>) got accepted at EMNLP 2025.</li>
<li><span class="news-date">Aug 2024</span>A paper (<a href="https://openaccess.thecvf.com/content/CVPR2024/papers/Park_MedBN_Robust_Test-Time_Adaptation_against_Malicious_Test_Samples_CVPR_2024_paper.pdf" target="_blank">MedBN</a>) got accepted at CVPR 2024.</li>
</ul>

## Selected Publications

<div class="selected-pubs">
{% assign selected = site.publications | where: "selected", true | sort: "slug" | reverse %}
{% for post in selected %}
  {% include archive-single.html %}
{% endfor %}
</div>
