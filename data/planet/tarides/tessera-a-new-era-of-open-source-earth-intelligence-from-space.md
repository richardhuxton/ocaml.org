---
title: 'Tessera: A New Era of Open-Source Earth Intelligence from Space'
description: Tessera is an open-source AI model, co-funded by Tarides, that turns
  petabytes of satellite imagery into Earth analysis you can run on a laptop.
url: https://tarides.com/blog/2026-10-29-tessera-a-new-era-of-open-source-earth-intelligence-from-space
date: 2026-10-29T00:00:00-00:00
preview_image: https://tarides.com/blog/images/green-satellite-image-1360w.webp
authors:
- Tarides
source:
ignore:
---

<p>We've all seen satellites drifting by at night, and wondered how many are actually up there. With how long they have been observing Earth, it is unsurprising that satellites have accumulated a vast library of images covering the globe. In a world increasingly focused on identifying and mitigating human impact on the natural world, these images constitute an untapped data resource.</p>
<p>Sadly, most conservation groups, field biologists, and local policymakers don't have a supercomputer on stand-by to help them trawl through that data and analyse trends. This is where the AI model <a href="https://geotessera.org/">Tessera</a>, co-funded by Tarides and developed by a team headed by the University of Cambridge, comes in.</p>
<h2>What is Tessera?</h2>
<p>Tessera is an open-source AI foundation model which can parse through petabytes of satellite data to reveal patterns and changes over time. Tessera lets scientists designate habitat maps and highlight the changes they want to monitor over time. It can be used to reveal environmental changes, such as crop patterns, habitat loss, and deforestation.</p>
<p>The software processes millions of satellite images from the <a href="https://www.esa.int/Applications/Observing_the_Earth/Copernicus/Tessera_AI_model_offers_accessible_way_to_view_Earth">European Space Agency</a> into a compact, easily usable format for scientists. The result is that, instead of needing a supercomputer, researchers have access to global-scale analyses from their laptops.</p>
<p>Tessera's algorithm learns to understand the 'fingerprint' of a landscape, knowing what it should look like, which allows it to fill in cloud-covered images with the missing values. For each 10-by-10-metre of land on Earth, the software creates an uninterpreted sequence of 128 numbers called 'embeddings'.  All of these embeddings are available on demand in the database GeoTessera.</p>
<p>A recent example comes from the Scottish Highlands, where a Cambridge-led team used aerial surveys to build detailed 3D maps showing the beginnings of large-scale native woodland recovery across the Cairngorms National Park. Tarides supported that work, and the data feeds into Tessera. You can read the <a href="https://www.cam.ac.uk/research/news/successful-woodland-expansion-in-scotlands-cairngorms-revealed-in-aerial-maps">University of Cambridge write-up</a>.</p>
<p>To learn more about the project, you can read the <a href="https://www.cst.cam.ac.uk/research/eeg/cambridge-ai-tool-promises-new-era-earth-intelligence-space">article published by the University of Cambridge</a>, the <a href="https://openaccess.thecvf.com/content/CVPR2026/papers/Feng_TESSERA_Temporal_Embeddings_of_Surface_Spectra_for_Earth_Representation_and_CVPR_2026_paper.pdf">open access paper</a>, and watch the <a href="https://www.youtube.com/watch?v=J-QLuX9xcsI">webinar on Tessera</a>.</p>
<h2>Tessera &amp; Tarides: A Natural Fit with our Work in Space</h2>
<p>We are proud to have sponsored Tessera's development, since it is open source and has no paywalls by design. As Tessera's co-lead, Anil Madhavapeddy, professor of planetary computing at Cambridge, puts it: the goal is to "democratise satellite data for people who lack massive computer resources, whether that's an underfunded conservation group here in the UK, field biodiversity researchers in the majority world, or planners figuring out climate adaptation strategies in a rapidly warming world."</p>
<p>Tarides is dedicated to building secure and sustainable software with a positive impact, and supporting the Tessera project is a natural fit. It also complements our work on satellite software. Our sister company, Parsimoni, has been putting OCaml and MirageOS into orbit through <a href="https://tarides.com/blog/2025-04-03-ocaml-in-space-spaceos-is-on-a-satellite/">SpaceOS</a>, and we've built <a href="https://tarides.com/blog/2024-11-27-irmin-on-mirageos-introducing-the-notafs-file-system/">Notafs</a>, a filesystem that lets Irmin run on MirageOS unikernels aboard satellites. Earlier this year, we also kicked off our part in the <a href="https://tarides.com/blog/2025-05-30-ceos-project-kick-off-using-satellites-to-survey-the-earth/">CEOS project</a>, building next-generation Earth observation satellites.</p>
<p>Tessera is a piece of that same puzzle; it's one thing to get reliable, secure software into orbit and collect data, and another to make sense of what that data is telling us once it's back on the ground. Whether it's the satellite hardware and unikernels collecting the imagery, or a foundation model turning that imagery into insight for a conservation group with no cloud budget, the focus for us is the same: open tools should make space and Earth science more accessible, not less.</p>
<h2>What's Next &amp; Stay in Touch</h2>
<p>Explore Tessera's embeddings yourself at <a href="https://geotessera.org/">geotessera.org</a>, and share your thoughts with the team behind the model.</p>
<p>Stay in touch with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up to our mailing list to stay updated on our latest projects. We look forward to hearing from you!</p>

