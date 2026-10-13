# Seedream 5.0 Flash API (`seedream-5-0-flash`) — API ゲートウェイ (日本語)

<!-- conv-kit:v1 -->

<p align="center">
  <img src="assets/badges/price.svg" alt="observed unit price"> <img src="assets/badges/billing.svg" alt="billing model"> <img src="assets/badges/compat.svg" alt="OpenAI-compatible endpoint">
</p>

<p align="center">
  <img src="assets/01-product-hero-perfume.jpg" width="820" alt="Sample render via the APIMart image API">
</p>

> **$0.0137 per image** — one price for 1K, 1.5K, 2K and exact pixel sizes — one OpenAI-compatible endpoint at `https://api.apimart.ai/v1`, no monthly plan required. *(observed 2026-10-12)*

**[APIキーを取得](https://go.apimart.ai/k-b83d96)** · **[最新の料金](https://go.apimart.ai/k-de2698)** · **[モデルページ](https://go.apimart.ai/k-74fb9c)** · [⚡ 60-second quickstart](#quickstart)

**Why teams call Seedream 5.0 Flash (`seedream-5-0-flash`) through APIMart**

- **One key, entire catalog.** The same `https://api.apimart.ai/v1` base URL and `Authorization` header reach Seedream 5.0 Flash and 300+ other image, video and language models — switch the `model` field, not your client.
- **$1 minimum, pay as you go.** No subscription and no prepaid plan to size up front: top up from $1 and spend it on calls. There is no free quota to burn through first, so the price in this table is the price you pay.
- **The charge comes back in the response.** Every completed task reports the amount billed (`cost` / `credits_cost`), so a spend number is read per call instead of guessed at month end. A task marked `failed` is refunded.
- **Async by design.** Submit, take the `task_id`, poll `GET /v1/tasks/{id}` — batching and retries are ordinary queue work, not a bespoke integration.

<!-- /conv-kit:v1 -->

Seedream 5.0 Flash：OpenAI 互換の `base_url` と 1 つのキーで、Seedream 5.0 Flash（`seedream-5-0-flash`）と 300+ の画像・動画・言語モデルを呼び出せます。USD 決済・従量課金・最低 $1 から。

Seedream 5.0 Flash is ByteDance's fast, lower-cost text-to-image model, served on APIMart as `seedream-5-0-flash`. It generates cinematic 1K, 1.5K and 2K images — or exact pixel sizes — at one per-image price, with strong text rendering, multi-reference style fusion, and unified generation plus editing.

## Model id and routes

| Route | `model` value | Billing | Images per task | Notes |
| --- | --- | --- | --- | --- |
| Create | `seedream-5-0-flash` | per image, one price for every size | 1 image | text-to-image |
| Edit | `seedream-5-0-flash` | per image, one price for every size | 1 image | reference-guided generation and editing |

Endpoint: `POST https://api.apimart.ai/v1/images/generations` (OpenAI-compatible), then poll `GET /v1/tasks/{task_id}`.
Result links expire **72 hours** after completion.

## Pricing

<!-- pricing:model:start -->
| Output | List price | Effective price |
| --- | --- | --- |
| default | $0.0375 | $0.03 |
| 4K | $0.05 | $0.04 |

The token-billed official route bills per million tokens instead: `text_input` $1.60, `cached_text_input` free, `image_input` $1.60, `cached_image_input` free, `text_output` $9.60, `image_output` $96.00.
<!-- pricing:model:end -->

Generation is asynchronous — poll the task until it completes. Prices are a snapshot; the [最新の料金](https://go.apimart.ai/k-de2698) page and [`data/model.json`](data/model.json) are refreshed by CI, and a completed task reports the exact amount in its `cost` field.

## Request parameters

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `model` | string | required | `seedream-5-0-flash` |
| `prompt` | string | required | natural language with structure (Subject + Style + Details); put literal text in double quotes |
| `size` | string | `1:1` | aspect ratio, or exact pixel sizes such as `2048x1024` |
| `resolution` | string | `1K` | `1K`, `1.5K`, `2K` — all sizes use the same image price |
| `image_urls` | string[] | — | up to 10 reference images to guide style, character appearance and composition |
| `mode` | string | `Create` | `Create` or `Edit` |
| `output_format` | string | `jpeg` | output format for the returned file |
| `prompt_mode` | string | `off` | Prompt Mode; `watermark` and content moderation toggles are also exposed |

**Reference images.** Up to **10** reference images guide style, character appearance and composition.

Supported aspect ratios: aspect-ratio presets plus exact pixel sizes (e.g. `2048x1024`).

## Quickstart

```bash
curl --request POST --url https://api.apimart.ai/v1/images/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" --header 'Content-Type: application/json' \
  -d '{"model":"seedream-5-0-flash","prompt":"A bamboo forest path under moonlight","size":"1:1","resolution":"1K"}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

created = requests.post(f"{BASE}/images/generations", headers=HEADERS, timeout=60, json={
    "model": "seedream-5-0-flash", "prompt": "A bamboo forest path under moonlight",
    "size": "1:1", "resolution": "1K",
}).json()
task_id = created["data"]["id"]

while True:
    task = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json()["data"]
    if task["status"] in ("completed", "failed"):
        break
    time.sleep(5)
print(task.get("cost"), task["result"]["images"][0]["url"])
```

```javascript
const res = await fetch("https://api.apimart.ai/v1/images/generations", {
  method: "POST",
  headers: { Authorization: `Bearer ${process.env.APIMART_API_KEY}`, "Content-Type": "application/json" },
  body: JSON.stringify({ model: "seedream-5-0-flash", prompt: "A bamboo forest path under moonlight",
                          size: "1:1", resolution: "1K" }),
});
const { data } = await res.json();      // data.id is the task id — poll /v1/tasks/<id>
```

Runnable versions: [`examples/`](examples). The task lifecycle is `pending → processing → completed | failed`.

## Sample outputs

Each row is one API call. Costs follow the price table above.

| Output | Recipe | Ratio | Cost | Prompt |
| --- | --- | --- | --- | --- |
| <img src="assets/01-product-hero-perfume.jpg" width="220" alt="Sample render via the APIMart image API"> | Product / e-commerce | 1:1 | $0.0137 | `Studio hero shot of a frosted glass perfume bottle on a wet black stone slab, single softbox from the left, faint mist, deep charcoal background, crisp label text, commercial product photography` |
| <img src="assets/02-food-overhead-ramen.jpg" width="220" alt="Sample render via the APIMart image API"> | Food photography | 4:3 | $0.0137 | `Overhead flat lay of a spicy miso ramen bowl with soft boiled egg, nori and scallions, dark ceramic table, chopsticks resting on the rim, natural window light, editorial food photography` |
| <img src="assets/03-cinematic-street-rain.jpg" width="220" alt="Sample render via the APIMart image API"> | Cinematic still | 16:9 | $0.0137 | `Rain soaked Kyoto alley at night, paper lantern reflections on wet stone, a lone figure with a transparent umbrella walking away, cinematic 35mm still, shallow depth of field, film grain` |
| <img src="assets/04-architectural-dusk.jpg" width="220" alt="Sample render via the APIMart image API"> | Architecture | 16:9 | $0.0137 | `Minimalist concrete villa on a cliff at dusk, warm interior light, infinity pool reflecting a violet sky, architectural photography, 24mm perspective, ultra sharp` |
| <img src="assets/05-fashion-editorial-studio.jpg" width="220" alt="Sample render via the APIMart image API"> | Fashion portrait | 3:4 | $0.0137 | `Editorial fashion portrait of a model in an oversized ivory wool coat, seamless light grey studio backdrop, crisp high key lighting, medium format detail, calm expression` |
| <img src="assets/06-illustration-layered-paper.jpg" width="220" alt="Sample render via the APIMart image API"> | Illustration | 1:1 | $0.0137 | `Layered paper cut illustration of a mountain lake at sunrise, five depth layers, soft pastel palette, subtle drop shadows, art print composition, clean vector edges` |
| <img src="assets/07-infographic-dashboard.jpg" width="220" alt="Sample render via the APIMart image API"> | Design / UI | 16:9 | $0.0137 | `Flat vector dashboard panel showing three gauge dials and abstract latency curves, muted blue and sand palette, generous white space, crisp geometric cards, no placeholder text` |
| <img src="assets/08-macro-botanical-detail.jpg" width="220" alt="Sample render via the APIMart image API"> | Macro nature | 3:2 | $0.0137 | `Macro photograph of a dew covered dandelion seed head against deep black background, focus stacked detail, iridescent droplets, studio lighting` |

Sources and costs are also in [`data/samples.json`](data/samples.json).

## FAQ

**What is the Seedream 5.0 Flash model id?**

`seedream-5-0-flash`. See the route table above for the exact values this model accepts.

**How is a Seedream 5.0 Flash image billed?**

List price $0.017143; the effective rate reflects the standard discount tier.

**How long do result links last?**

72 hours after completion — read `expires_at` and copy the files out.

**What happens if a task fails?**

A task marked `failed` is refunded and reports `cost = 0`. A network error while polling status does not by itself mean the task failed — re-query the task id.

## Attributed links (how this repository is measured)

| Purpose | Attributed link | Target |
| --- | --- | --- |
| モデルページ | <https://go.apimart.ai/k-74fb9c> | `apimart.ai/ja/model/seedream-5-0-flash` |
| 最新の料金 | <https://go.apimart.ai/k-de2698> | `apimart.ai/ja/pricing` |
| APIキーを取得 | <https://go.apimart.ai/k-b83d96> | `apimart.ai/ja/keys` |

Outbound APIMart links are minted through the promo link API; hand-made tracking parameters are rejected by
`tools/check_links.py` in CI.

## Disclosure

Seedream 5.0 Flash is a third-party model served through APIMart; this repository documents how to call it and publishes
model ids and prices, and does not claim official status. Model names, prices and documentation belong to their
respective owners. The renders in this README were produced through the APIMart image API and are shown as examples
of the API's output.

## Repository map

```text
README.md             model id, routes, pricing, parameters, quickstart, samples, FAQ
data/model.json       the pricing record for this model (CI-refreshed)
data/samples.json     prompt recipes used for the sample renders
tools/check_links.py  attribution guard
examples/             curl, Python and JavaScript clients
assets/               sample renders (JPEG, resized for the README)
```

## License

MIT — see [LICENSE](LICENSE).
