---
title: '.plan-26-38: Near, far, wherever you are, Tessera infers on'
description: Working through dClimate's wall-to-wall v1.1 Tessera embeddings, and
  finding threatened species near you with the Dash of Life.
url: https://anil.recoil.org/notes/2026w38
date: 2026-09-20T00:00:00-00:00
preview_image: https://anil.recoil.org/images/tessera-climate-validate-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<h2><a href="https://anil.recoil.org/news.xml#tessera-now-has-wall-to-wall-embeddings-over-nine-years" class="anchor" aria-hidden="true"></a>Tessera now has wall to wall embeddings over nine years!</h2>
<p>The big news this week is that Tessera community member <a href="https://blog.dclimate.net/mapping-a-changing-planet-tesseras-global-earth-observation-embeddings-now-openly-available-on-aws/">dClimate announced</a> that global wall-to-wall v1.1 <a href="https://anil.recoil.org/projects/tessera">Tessera</a> embeddings are now available on AWS for 2017-2025!
This marks the first time that we have complete embeddings coverage for so many years for any of our models, which is a giant milestone for the project. This inference was possible thanks to a grant that dClimate received from <a href="https://aws.amazon.com/opendata/">AWS Open Data</a>.</p>
<p>This also highlights just how cool working in the open is, since dClimate reimplemented the Tessera inference engine to specialise it to high-performance processing on cloud infras. We've been exchanging tips since May on our <a href="https://eeg.zulipchat.com/#narrow/channel/587016-Embeddings-Generation/topic/Open.20Source.20cloud.20TESSERA.20repo/with/625660416">Zulip</a> and clarifying various details in their <a href="https://github.com/dClimate/tessera-embeddings">codebase</a>.  Having an independent validation of our inference methods is as much of a big deal as the embeddings themselves, given the importance of these to such a <a href="https://anil.recoil.org/notes/geotessera-a-year-on">variety of downstream tasks</a> from which important policy decisions may be driven.</p>
<p><a href="https://www.tunbury.org/">Mark Elvers</a> and I have been working this week on integrating all this smoothly into <a href="https://anil.recoil.org/notes/geotessera-python">GeoTessera</a> so that our users can take advantage of the new embeddings. Here are some of my working notes on the topic on <a href="https://anil.recoil.org/news.xml#cross-validating-the-new-v11-embeddings-and-sources">cross-validating the two v1.1 runs</a>, <a href="https://anil.recoil.org/news.xml#checking-equivalent-coverage-across-the-two">checking their coverage</a>, <a href="https://anil.recoil.org/news.xml#converting-from-icechunk-to-zarr-v3">converting Icechunk to Zarr v3</a> and <a href="https://anil.recoil.org/news.xml#weaving-the-matrix-of-embeddings-into-a-client">weaving the matrix into GeoTessera</a>. I haven't had the time to cover my whole week, but here's also a dashboard of <a href="https://anil.recoil.org/news.xml#dash-of-life-finds-threatened-species-near-you">threatened species near you</a> and some <a href="https://anil.recoil.org/news.xml#fun-photos">fun photos</a>!</p>
<h3><a href="https://anil.recoil.org/news.xml#cross-validating-the-new-v11-embeddings-and-sources" class="anchor" aria-hidden="true"></a>Cross validating the new v1.1 embeddings and sources</h3>
<p>The first thing we did was to cross-validate the embeddings to make sure there
weren't big differences in performance.  One notable change resulting from the inference
mechanism running on Amazon is that they switched to a slightly different satellite
data source.</p>
<p>While both are derived from the same <a href="https://sentiwiki.copernicus.eu/web/s2-products">ESA Sentinel-2 L2A products</a>, they aren't
quite identical. On AWS, Element 84 runs "<a href="https://element84.com/earth-search/">Earth Search</a>"
over its own <a href="https://registry.opendata.aws/sentinel-2-l2a-cogs/">cloud-optimised GeoTIFFs</a>,
whereas the <a href="https://planetarycomputer.microsoft.com/dataset/sentinel-2-l2a">Planetary Computer</a>
maintains its own archive and <a href="https://github.com/microsoft/PlanetaryComputer/discussions/369">until mid-2024 ran Sen2Cor directly</a>
to produce the L2A data.</p>
<p>The inventories therefore differ subtly; e.g. one user
<a href="https://github.com/microsoft/PlanetaryComputer/discussions/371">found ~98k scenes on Element 84 vs ~76k on MPC</a>
for the same UK fields and dates.
The data formats also need different handling due to encoding differences. For example, ESA adds a +1000 offset to every reflectance value so that -ve reflectance can be represented. However, AWS removes this offset in the Element84 dataset, but MPC serves the values raw.
dClimate <a href="https://github.com/dClimate/tessera-embeddings/blob/main/context_docs/decisions/020-boa-offset-applies-to-every-valid-dn.md">found that</a>
getting it wrong makes every pixel silently too bright or too dark by the same amount!</p>
<p>In order to test this all, we did a few lightweight tests. Our usual 'go to' has been requesting <a href="https://patball1.github.io">James G. C. Ball</a> to run his <a href="https://anil.recoil.org/papers/2026-tessera-trentino">Trentino tree species mapping</a> but he was on a well deserved vacation this week. So instead <a href="https://www.tunbury.org/">Mark Elvers</a> used the <a href="https://esa-worldcover.org/en">ESA WorldCover map</a> to label a 100m grid over Cayenne, and then used 80% of the points to fit a linear classifier and predict classes. Both performed identically.</p>
<p><img src="https://anil.recoil.org/images/tessera-climate-validate-1.webp" alt="%c" title="WorldCover 2021 labels over Cayenne, linear probe predictions from the Cambridge (MPC) and dClimate (AWS) embeddings, and where they disagree (credit: Mark Elvers)"></p>
<p>Then <a href="https://toao.com">Sadiq Jaffer</a> ran his <a href="https://toao.com/blog/earth-observation-budget-solar-farms-tiny-model">tiny solar farm segmentation model</a> and also found equivalent performance.
However, one important <a href="https://eeg.zulipchat.com/#narrow/channel/527258-Tessera/topic/geotessera.200.2E11.20being.20prepared/near/625758530">finding</a> from Sadiq is that we cannot use the differently inferred v1.1 embeddings in the <em>same</em> analysis, since both were inferred from different sources:</p>
<div role="region"><table>
<tbody><tr>
<th>train source</th>
<th>test source</th>
<th>IoU</th>
<th>Dice</th>
<th>Precision</th>
<th>Recall</th>
</tr>
<tr>
<td>cambridge</td>
<td>cambridge</td>
<td>0.600 ± 0.004</td>
<td>0.750</td>
<td>0.632</td>
<td>0.922</td>
</tr>
<tr>
<td>dclimate</td>
<td>dclimate</td>
<td>0.616 ± 0.010</td>
<td>0.763</td>
<td>0.642</td>
<td>0.941</td>
</tr>
<tr>
<td>cambridge</td>
<td>dclimate</td>
<td>0.493 ± 0.047</td>
<td>0.659</td>
<td>0.535</td>
<td>0.903</td>
</tr>
<tr>
<td>dclimate</td>
<td>cambridge</td>
<td>0.540 ± 0.019</td>
<td>0.701</td>
<td>0.627</td>
<td>0.819</td>
</tr>
</tbody></table></div><p>Sadiq's checks above show that each set of v1.1 embeddings works equally well on its own, but a model trained on one
and tested on the other loses between 0.06 and 0.11 IoU, and its variance rises
sharply. Although both runs used the same v1.1 model, the embeddings are computed
from different copies of the input imagery, with different scene inventories and
offset handling.</p>
<p>We will therefore keep <code>1.1-cam</code> and <code>1.1-dclimate</code> as separate variants in GeoTessera,
and you should be careful to pick one and stick with it for any given analysis (but swapping
them wholesale should make no practical difference).</p>
<h3><a href="https://anil.recoil.org/news.xml#checking-equivalent-coverage-across-the-two" class="anchor" aria-hidden="true"></a>Checking equivalent coverage across the two</h3>
<p>One persistent problem with most GeoFMs is that some areas of the world have many fewer observations than others, and so the quality of inference can vary if the number of annual satellite observations is very low. Our <a href="https://anil.recoil.org/papers/2025-tessera">Tessera v1 paper</a> ran ablations to show that performance dropped sharply if n&lt;10 for S2.</p>
<p>The dClimate folks decided to take the route of not generating embeddings for those areas for which we have low coverage, to keep a consistent quality bar. <a href="https://eeg.zulipchat.com/#narrow/channel/587016-Embeddings-Generation/topic/missing.20dclimate.20v1.2E1.20embeddings/near/625739045">Robert showed</a> that this is only a very small percentage of areas, so most users should hopefully never notice. Kristian Bodolai from Space Intelligence <a href="https://github.com/ucam-eo/geotessera/issues/424">started a discussion</a> about what to do with these embeddings, and apparently they are still getting excellent results <a href="https://eeg.zulipchat.com/#narrow/channel/587016-Embeddings-Generation/topic/missing.20dclimate.20v1.2E1.20embeddings/near/625756328">doing palm oil classification</a> even in those low coverage areas, possibly thanks to the S1 coverage still holding up the quality of the inferred embedding.</p>
<p>We'll make more progress on this next week, and I hope to reach consensus on what to do.
Even though the 'Cambridge embeddings' do cover those areas, remember that we
can't mix them safely in the same task.</p>
<h3><a href="https://anil.recoil.org/news.xml#converting-from-icechunk-to-zarr-v3" class="anchor" aria-hidden="true"></a>Converting from Icechunk to Zarr v3</h3>
<p>The <a href="https://sustainabilityexchange.amazon.com/">Amazon Sustainability Data Initiative</a> and the <a href="https://opendata.aws/">AWS Open Data Sponsorship Program</a> that sponsored the inference also cover the hosting of the result, which dClimate publish as an <a href="https://icechunk.io">Icechunk</a> repository. Icechunk adds database-style transactions to Zarr, which <a href="https://github.com/dClimate/tessera-embeddings/blob/main/docs/global-store.md">matters at this scale</a> as a truncated write to a normal Zarr hierarchy on object storage will be undetectable.  <a href="https://www.tunbury.org/">Mark Elvers</a> and I have been converting it over to plain Zarr v3 on <a href="https://source.coop/tessera/tessera">Source Coop</a> to allow non-Icechunk clients (like my OCaml code!) to access the data.</p>
<p>Mark has been running the conversion on <a href="https://www.tunbury.org/2026/09/21/week-38/#icechunk-to-zarr-conversion">AWS Fargate Spot as we did before</a>, so it all runs as containers that have been churning through the world at around 113 GB/minute, and is about two-thirds done at the time of writing.</p>
<p>One possible screwup I might have made during this conversion is picking too small a chunk size for the Zarr. The Source Coop stores keep the <code>(1,128,32,32)</code> inner chunks inside <code>4096×4096</code> shards that I <a href="https://anil.recoil.org/notes/tessera-embeddings-convention">chose back in March</a>. The dClimate store uses much larger <code>256×256</code> inner chunks inside <code>2048×2048</code> shards, and so a single pixel fetch is about <a href="https://github.com/dClimate/tessera-embeddings/blob/main/docs/global-store.md#what-a-read-costs">8.65 MB of traffic</a>.</p>
<p>In practice, scattered point lookups are expensive in the dClimate store, but windowed reads are more expensive per pixel in my Source Coop store.
There's therefore a tradeoff between streaming into a browser and streaming for the cloud, which we've been <a href="https://eeg.zulipchat.com/#narrow/channel/587016-Embeddings-Generation/topic/zarr.20v3.20chunk.20sizes/near/625720009">discussing on Zulip</a>. I suspect we'll just settle on different chunk sizes for the Icechunk and Zarr v3 stores in the end to accommodate different clients. Source Coop feels more appropriate for the mobile use case due to the Cloudflare edge caching it provides. More research needed!</p>
<h3><a href="https://anil.recoil.org/news.xml#weaving-the-matrix-of-embeddings-into-a-client" class="anchor" aria-hidden="true"></a>Weaving the matrix of embeddings into a client</h3>
<p>I've also been <a href="https://github.com/ucam-eo/geotessera/pull/422">teaching</a> GeoTessera to handle this matrix of models.
In the upcoming version, the v1.1 becomes the default model now that we have so much coverage. While the Zarr conversion is ongoing, streamed reads go straight to the dClimate
Icechunk store.</p>
<div role="region"><table>
<tbody><tr>
<th>version</th>
<th>variant</th>
<th>format and home</th>
<th>years</th>
<th>status</th>
</tr>
<tr>
<td><code>1.0</code></td>
<td><code>vultr</code></td>
<td>NPY + Zarr on <a href="https://data.source.coop/tessera/tessera">source.coop</a> (<code>npy/v1/</code>, <code>zarr/v1</code>)</td>
<td>2017–2025</td>
<td>First production line</td>
</tr>
<tr>
<td><code>1.1</code></td>
<td><code>cambridge</code></td>
<td>NPY + Zarr on source.coop (<code>npy/v1.1-cam/</code>, <code>zarr/v1.1</code>)</td>
<td>2015–2025</td>
<td>Cambridge deployment that's the NPY-tile default, but thinner coverage</td>
</tr>
<tr>
<td><code>1.1</code></td>
<td><code>dclimate</code></td>
<td>Icechunk on AWS S3</td>
<td>2017–2025</td>
<td>The new global run and the streamed default; no NPY tiles at all</td>
</tr>
<tr>
<td><code>2.0</code></td>
<td><code>2B-L~beta1</code></td>
<td>NPY + Zarr on source.coop (<code>v2-2B-L~beta1/</code>)</td>
<td>2017–2025</td>
<td>v2 beta, 2B parameters, L size. Experimental</td>
</tr>
<tr>
<td><code>2.0</code></td>
<td><code>2B-L~beta2</code></td>
<td>NPY + Zarr on source.coop (<code>v2-2B-L~beta2/</code>)</td>
<td>2017–2025</td>
<td>Second v2 beta run. Experimental but poor temporal stability</td>
</tr>
</tbody></table></div><p><a href="https://mynameismwd.org">Michael Dales</a> also reported <a href="https://github.com/ucam-eo/geotessera/issues/417">occasionally unusable source.coop performance</a>, which I'm tracing to some possible instability in the Source Coop Rust proxy. More on this when I investigate next week too!</p>
<h2><a href="https://anil.recoil.org/news.xml#dash-of-life-finds-threatened-species-near-you" class="anchor" aria-hidden="true"></a>Dash of Life finds threatened species near you</h2>
<p><a href="https://shaneweisz.com">Shane Weisz</a>'s <a href="https://dashoflife.org">Dash of Life</a> from our <a href="https://anil.recoil.org/projects/enki">biodiversity mapping project</a> now has a <a href="https://dashoflife.org/near-me">near me page</a> that lists the threatened species records around a point, along with the corresponding threats that their <a href="https://www.iucnredlist.org">IUCN Red List</a> assessments cite. I added a button to zoom straight into your current location (<a href="https://github.com/shaneweisz/redlist-dashboard/pull/555">PR #555</a>) so you can learn more about your local region with a quick browser bookmark.</p>
<p>Within 10 km of Cambridge, the list shows the <a href="https://en.wikipedia.org/wiki/Common_pochard">common pochard</a>, <a href="https://en.wikipedia.org/wiki/Aesculus_hippocastanum">horse chestnut</a> and <a href="https://en.wikipedia.org/wiki/European_turtle_dove">European turtle dove</a> as vulnerable, the <a href="https://en.wikipedia.org/wiki/European_rabbit">European rabbit</a> as endangered, and the <a href="https://en.wikipedia.org/wiki/European_eel">European eel</a> as critically endangered!</p>
<p><img src="https://anil.recoil.org/images/dashoflife-near-me.webp" alt="%c" title="Threatened species with GBIF records within 10 km of Cambridge on the Dash of Life"></p>
<h2><a href="https://anil.recoil.org/news.xml#fun-photos" class="anchor" aria-hidden="true"></a>Fun photos</h2>
<p><a href="https://web.eecs.umich.edu/~comar/">Cyrus Omar</a> has made his way to Cambridge to start his sabbatical, and <a href="http://carlhenrik.com/">Carl Henrik Ek</a> and I rolled out the red carpet at the Mill for him!</p>
<p><img src="https://anil.recoil.org/images/cyrus-visit26-1.webp" alt="%c" title="Cyrus Omar is convinced that the Mill is the only place in Cambridge we hang out"></p>
<p>And I enjoyed seeing some gorgeous bikes and races at the <a href="https://www.goodwood.com/motorsport/goodwood-revival/">Goodwood Revival</a>, making the most of the September good weather!</p>
<p><img src="https://anil.recoil.org/images/goodwood-1.webp" alt="%c" title="Some amazing bikes...">
<img src="https://anil.recoil.org/images/goodwood-2.webp" alt="%c" title="...amazing cars...">
<img src="https://anil.recoil.org/images/goodwood-3.webp" alt="%c" title="...and cutthroat wheel-to-wheel races!"></p><h1>References</h1><ul><li>Feng et al (2026). TESSERA: Temporal Embeddings of Surface Spectra for Earth Representation and Analysis. <a href="https://doi.org/10.48550/arXiv.2506.20380" target="_blank"><i>10.48550/arXiv.2506.20380</i></a></li>
<li>Ball et al (2026). Geospatial foundation models enable data-efficient tree species mapping in temperate mountain forests. Elsevier BV. <a href="https://doi.org/10.1016/j.srs.2026.100466" target="_blank"><i>10.1016/j.srs.2026.100466</i></a></li>
<li>Madhavapeddy (2026). TESSERA now supports the Zarr geo-embeddings convention proposal. <a href="https://doi.org/10.59350/c3hrq-zsx02" target="_blank"><i>10.59350/c3hrq-zsx02</i></a></li>
<li>Madhavapeddy (2025). GeoTessera Python library released for geospatial embeddings. <a href="https://doi.org/10.59350/7hy6m-1rq76" target="_blank"><i>10.59350/7hy6m-1rq76</i></a></li></ul>
