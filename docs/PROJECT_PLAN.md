# SeasonVision AI — build plan

Updated: 13 November 2026. Status: agreed constraints incorporated; implementation plan v1. No model execution or application implementation verified. Architecture below is the recommended route under these constraints; model performance remains to be tested.

## User constraints and working decisions

- Pretrained inference is provisionally allowed for planning, at the user's request. Professor confirmation is pending. Continue preparation on this assumption; if training is required, revise scope before claiming course compliance.
- Zero spending: no paid APIs, GPU subscriptions or paid hosting in the plan.
- User has used Colab before; free notebook compute is a candidate, not guaranteed availability. A friend's better laptop can be borrowed temporarily; its GPU/VRAM/RAM are unknown.
- Browser-based app preferred; internet/offline rules not specified. A web interface can run locally and does not require public hosting.
- Four team members. User leads implementation and technical understanding; others can handle report and supporting work. Exact role allocation remains tentative.
- Balanced ambition: meaningful GenAI content and visible contributions, with bounded complexity and fallback paths.
- Study focus is the GenAI pipeline actually implemented. Include a short Python code walkthrough; avoid a full unrelated programming syllabus.
- Submission/viva deadline has not yet been supplied. Sequence work by milestones rather than inventing dates.

## Product and success criteria

Create seasonal retail campaign imagery from an actual uploaded product photo. The student must be able to explain every deployed stage and distinguish team code from pretrained models and upstream workflows.

Recommended MVP: upload product → inspect/correct foreground mask → choose season and style → inspect/edit generated prompt → generate background → composite original product → compare and download image plus generation metadata.

Start with Diwali, Christmas and Summer; two styles (minimal and festive); one image per request; square composition first. These are proposed scope limits. Add portrait/landscape only after resizing and masks work correctly. Keep labels/logos from the original product. Use deterministic text overlays if campaign headlines are later added.

Success means a repeatable same-product demonstration, acceptable seasonal relevance, clear failure reporting and an explainable code path. A polished interface without real image inference does not pass.

## Recommended architecture and decision gates

Recommended v1: Python + Gradio + Pillow + Hugging Face Diffusers/PyTorch, with one pretrained segmentation component. Keep the model loaded once per process and handle one generation at a time. Use the same pipeline functions from the research notebook and app. Final segmentation checkpoint is selected during mask testing; InSPyReNet is the supplied workflow's reference implementation.

Shopify/Furkan and Pixaroma are implementation references, not a commitment to install every upstream component. SDXL is the model family; Shopify's background-replacement application is one system built around it. The supplied SDXL JSON uses a Juggernaut checkpoint, while our first clean baseline below is a different SDXL inpainting checkpoint. Do not call our code a reproduction until its behavior has been compared.

Use ComfyUI only as a reference/prototyping tool initially. If a necessary source-workflow behavior cannot be reproduced reasonably with the Python pipeline, document the evidence before changing the backend. Avoid two production inference stacks.

First baseline candidate: `diffusers/stable-diffusion-xl-1.0-inpainting-0.1`, which accepts an image, mask and text prompt [S7]. Pin a tested model revision and package versions after the first successful run. Model card limitations and license must travel with the implementation. This is a candidate, not a demonstrated result. SD 1.5 inpainting is the smaller fallback. Flux, GGUF/T5, generative upscaling and relighting are outside v1; ControlNet is added only if a measured failure justifies it.

This recommendation favors learning, product preservation and reproducibility over adding every model. No price, cloud access, speed or memory fit is assumed. Offloading can reduce GPU memory use but transfers work/data and can slow inference [S2].

Compute plan: use free Colab for interactive notebook experiments. Google's FAQ says free managed runtimes restrict bypassing the notebook to interact primarily through a web UI, and GPU availability varies [S6]. Therefore do not use free Colab as our app hosting/API server. Run the Gradio app locally on a tested borrowed laptop for the main demonstration, after obtaining actual specifications. Once dependencies and weights are cached, rehearse with internet disconnected if offline operation is needed. The user's GTX 1650 remains suitable for development and a bounded smaller-model test, not an assumed SDXL host.

Execution gate:

1. Establish a notebook SDXL baseline on an available Colab GPU or suitable borrowed laptop. If Colab GPU is unavailable, do CPU preprocessing/UI work while arranging the friend's machine; never rely on repeated free-quota retries.
2. Test the intended demo laptop early. Start with one product and record cold start, warm generation time, memory and quality. After two focused attempts at the same blocker, revisit model/setup rather than continue indefinitely.
3. If only the user's laptop is available, test a smaller SD 1.5 configuration with appropriate memory settings. Accept it only if measured product/season quality and runtime are usable; do not promise 4 GiB compatibility in advance.
4. If no machine can generate adequately, preserve notebook results and disclose the compute blocker. A saved-results gallery/recorded demonstration is a labelled backup, not a claim of live generation. Confirm assessment acceptability if live inference cannot be supplied.
5. Before viva, rehearse one cold start and three consecutive generations on the actual demonstration machine. Keep cached weights, pinned environment, representative inputs, saved outputs and a short recording. Do not substitute cached images silently.

## Source workflow findings

Files are under `refrences/pixaroma-background-replacement/`; preserve originals.

| File | Contents actually inspected | Implication |
|---|---|---|
| `1_Flux Dev Replace Complex Background.json` | InSPyReNet extraction, inverted mask, Flux Q8, T5 Q8, CLIP-L, VAE, depth preprocessor, ControlNet, crop/stitch | Close reference; multiple model dependencies |
| `2_Flux Dev Replace Simple or White Background.json` | Same generation stages plus reference-background composite | Useful for preparing a starting scene |
| `SDXL Inpaint Replace Complex Background.json` | Juggernaut XL Version X Inpaint filename, ControlNet filename, InSPyReNet, depth and crop/stitch | SDXL reference candidate, not tested compatibility |
| `Image Upscaler.json` | Flux refinement and NMKD-Siax upscaling | Optional; can alter product details |

Generation nodes are saved in mode 2 (disabled); input nodes reference original/pasted image filenames. Extraction mask is saved and separately loaded by the generation graph: automate that connection in the final pipeline. These are UI graph exports, not immediately deployable app code. Identify custom-node repositories and pinned versions before execution; exact provenance is not established by node names alone.

Source settings are starting observations, not selected final settings: 1024-square; Flux 30 steps, guidance node 3.5, sampler CFG 1, denoise about .95; SDXL 30 steps, CFG 7, denoise .8. Flux guidance and SDXL CFG should not be taught as interchangeable.

## Build phases and evidence required

| Phase | Build work | Exit evidence | Parallel learning |
|---|---|---|---|
| 0. Requirements | Carry forward zero budget and provisional pretrained permission; collect deadline and demo-laptop specs | Decision record with unresolved items visible | T01–T03 |
| 1. Image preparation | Validate upload; preserve aspect ratio; extract foreground; show mask; provide correction | At least 3 product cutouts, including a difficult edge | T04–T06 |
| 2. One real generation | Run one pretrained inpainting candidate with fixed input, mask, prompt and seed | Saved output, actual model/version, settings, elapsed time, hardware and failure log | T07–T10 |
| 3. Preservation | Composite original foreground; correct mask edges; inspect duplicate silhouettes and lighting | Same-product comparison; interior pixels preserved where intended | T06, T11 |
| 4. Seasonal logic | Implement explicit season/style templates and editable prompt | Three seasons on one source product; saved prompts | T09, T12 |
| 5. App integration | Connect upload, mask review, generate, progress/error state, compare and download | Entire flow works without manual copying between graph nodes or Photoshop | T13–T14 |
| 6. Evaluation | Test varied products and multiple seasons; log quality, runtime and failures | Evaluation table and honest limitations | T15 |
| 7. Viva package | Diagram, contribution table, live demo, saved examples and oral questions | Student explains one request end to end without reading | All core topics |

After phase 3, add ControlNet only if baseline geometry/composition errors justify it. Compare identical inputs with and without it. After phase 6, consider better lighting, multiple candidates and aspect ratios. Generative upscaling is optional and must be checked for altered text.

## Planned modules (not existing files)

`app.py`: Gradio UI/events. `image_io.py`: validation, resizing and coordinate transforms. `segmentation.py`: foreground/mask. `prompts.py`: season/style templates. `generation.py`: one selected model/backend. `compositing.py`: preserve source pixels and blend boundaries. `records.py`: settings, outputs and evaluation metadata. `config.py`: explicit model/version/default settings.

Keep functions small enough to explain in viva. Introduce queues, databases or a separate web server only when a demonstrated requirement warrants them. No accounts, payments, video generation, training-from-scratch or multi-model orchestration in MVP.

## Evaluation and contribution evidence

Proposed matrix: 5 products × 3 seasons = 15 cases; include a labelled bottle/can, a box, and difficult reflective/transparent examples. Evaluate difficult examples as limitations if unsupported. Record season relevance, identity/logo readability, edges, shadow/lighting, artifacts, latency and retries. Use a documented human rubric (1–5 anchored as poor/partial/good); do not report it as an objective benchmark or model accuracy percentage.

Use baseline raw inpainting versus original-product compositing on the same inputs. If ControlNet is added, compare it separately. A fixed seed helps comparison but does not guarantee identical output across hardware/library changes. Preserve masks, prompts, model revision, package versions and selected outputs.

Explain team contributions as seasonal prompt logic, mask-to-generation automation, preservation, UI integration and evaluation when actually implemented. Credit Pixaroma/workflows and model authors. Training updates weights; our proposed application primarily performs inference. Instructor acceptance is a pending external dependency; user explicitly requested planning with pretrained inference for now. If fine-tuning becomes mandatory, define a small justified training experiment and revise compute/data scope; do not casually promise full SDXL training.

Suggested support allocation for four people: user owns implementation and end-to-end technical explanation; teammate A drafts report from verified evidence; teammate B helps curate test inputs and record rubric scores; teammate C prepares slides/demo logistics and attribution. Teammates still learn the pipeline for their viva. Final assignment of roles is the team's decision.

## Study-note maintenance rule

Personal notes are in gitignored `private-study/`. At every implemented phase update: topic status, exact module/function or graph node, why it exists, inputs/outputs, one real example, failure case and viva answer. Required study covers GenAI fundamentals, latent diffusion, text conditioning, inpainting, sampling parameters, segmentation/masks, compositing and evaluation. Python and Gradio get a bounded code-flow explanation. Keep unused Flux/ControlNet/upscaler concepts outside the active syllabus. Do not mark a lecture watched or a concept understood without the student's confirmation. Do not infer model architecture from a generic node name such as `UnetLoaderGGUF`.

## Sources checked / reference status

- S1: https://huggingface.co/docs/diffusers/using-diffusers/inpaint — masks, inpainting and preserving unmasked content; official guide checked 13 Nov 2026.
- S2: https://huggingface.co/docs/diffusers/optimization/memory — memory/offloading tradeoffs; official guide checked 13 Nov 2026.
- S3: https://github.com/Comfy-Org/ComfyUI/blob/master/script_examples/basic_api_example.py — official API integration example checked 13 Nov 2026; API execution is an option, not implemented.
- S4: https://gradio.app/docs/gradio/imageeditor — official editor component for mask correction; checked 13 Nov 2026.
- S5: https://huggingface.co/black-forest-labs/FLUX.1-dev — identifies Flux Dev as a rectified-flow transformer; checked 13 Nov 2026.
- S6: https://research.google.com/colaboratory/faq.html — free compute limits and web-UI restrictions; checked 13 Nov 2026.
- S7: https://huggingface.co/diffusers/stable-diffusion-xl-1.0-inpainting-0.1 — first baseline candidate model card; checked 13 Nov 2026.
- Tutorial: https://www.youtube.com/watch?v=_-uEw4f_sG4 — description and transcript previously inspected; workflow files inspected locally, no runtime test.
- Historical context: `ACADEMIC FOLDER/SEASONVISION_AI_PROJECT_HANDOFF.md` (local, ignored).
