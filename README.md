# Veo 3.1 API (`veo3.1-fast-ext`)

<!-- conv-kit:v1 -->

<p align="center">
  <img src="assets/badges/price.svg" alt="observed unit price"> <img src="assets/badges/billing.svg" alt="billing model"> <img src="assets/badges/compat.svg" alt="OpenAI-compatible endpoint">
</p>

<p align="center">
  <img src="assets/01-product-ad-ceramic-mug-thumb.jpg" width="820" alt="Sample video frame generated through APIMart">
</p>

> **from $0.064 per second** on the APIMart route — `https://api.apimart.ai/v1` OpenAI 호환 엔드포인트 하나면 되고, 월 구독 플랜은 필요 없습니다. *(2026-10-12 기준)*

**[API 키](https://go.apimart.ai/k-bcc772)** · **[가격](https://go.apimart.ai/k-886aed)** · **[모델 페이지](https://go.apimart.ai/k-97128a)** · [⚡ 60초 퀵스타트](#quickstart)

**팀들이 APIMart를 통해 Veo 3.1(`veo3.1-fast-ext`)을 쓰는 이유**

- **키 하나로 카탈로그 전체를.** 동일한 `https://api.apimart.ai/v1` base URL과 `Authorization` 헤더로 Veo 3.1(`veo3.1-fast-ext`)과 이미지·비디오·언어 모델 300+ 개에 접근할 수 있습니다. 바꾸는 건 `model` 필드뿐이고 클라이언트는 그대로입니다.
- **최소 $1, 종량제.** 구독도, 사전에 규모를 정해야 하는 선불 플랜도 없습니다. $1부터 충전해 호출한 만큼만 쓰면 됩니다.
- **과금 내역은 응답으로 돌아옵니다.** 완료된 모든 작업은 청구 금액(`cost` / `credits_cost`)을 보고하고, `failed`로 표시된 작업은 환불됩니다.
- **비동기 설계.** 제출하고 `task_id`를 받아 `GET /v1/tasks/{id}`를 폴링하면 끝입니다. 배치와 재시도는 별도 통합이 아니라 평범한 큐 작업입니다.

<!-- /conv-kit:v1 -->

**다른 이름:** 비오 3 · 베오 3

Veo 3.1 API on APIMart provides access to both veo3.1-fast and veo3.1-quality models, the newest generation of the Google Veo 3 API. Veo 3.1 is Google DeepMind's upgrade to Veo 3, offering enhanced audiovisual quality, native synchronized audio, and advanced cinematic controls for professional video creation.

## Model id and routes

| Route | `model` value | Billing | Notes |
| --- | --- | --- | --- |
| Text-to-video / image-to-video | `veo3.1-fast-ext` | per generated second, by resolution | see the [model page](https://go.apimart.ai/k-97128a) for the exact tiers |

Endpoint: `POST https://api.apimart.ai/v1/videos/generations`, then poll `GET /v1/tasks/{task_id}`.

## Pricing

<!-- pricing:model:start -->
| Resolution | List price | Effective price |
| --- | --- | --- |
| default | 0.175 | 0.14 |


<!-- pricing:model:end -->

Prices are a snapshot; the [pricing page](https://go.apimart.ai/k-886aed) and [`data/model.json`](data/model.json) are refreshed by CI, and a completed task reports the exact amount in its `cost` field.

## Request parameters

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `model` | string | required | the model id shown in the route table above |
| `prompt` | string | required | the shot description; camera and subject motion belong here |
| `duration` | integer | route default | clip length in seconds — billing is per generated second |
| `resolution` | string | route default | 480P / 720P / 1080P or the route’s own tiers |
| `size` / `aspect_ratio` | string | `16:9` | output framing |
| `image_urls` | string[] | — | first frame, first + last frame, or style references, where supported |
| `seed` | integer | — | fix the seed to reproduce a take |

The table lists the fields accepted by the APIMart `videos/generations` endpoint. Check the [model page](https://go.apimart.ai/k-97128a) for anything this model adds on top.

## Quickstart

```bash
curl --request POST --url https://api.apimart.ai/v1/videos/generations \
  --header "Authorization: Bearer $APIMART_API_KEY" --header 'Content-Type: application/json' \
  -d '{"model":"veo3.1-fast-ext","prompt":"A bamboo forest at sunrise, slow push-in","duration":5,"resolution":"720P","size":"16:9"}'
```

```python
import os, time, requests

BASE = "https://api.apimart.ai/v1"
HEADERS = {"Authorization": f"Bearer {os.environ['APIMART_API_KEY']}", "Content-Type": "application/json"}

created = requests.post(f"{BASE}/videos/generations", headers=HEADERS, timeout=60, json={
    "model": "veo3.1-fast-ext", "prompt": "A bamboo forest at sunrise, slow push-in", "duration": 5,
}).json()
task_id = created["data"]["id"]

while True:
    task = requests.get(f"{BASE}/tasks/{task_id}", headers=HEADERS, timeout=60).json()["data"]
    if task["status"] in ("completed", "failed"):
        break
    time.sleep(5)
print(task.get("cost"), task["result"]["videos"][0]["url"])
```

Runnable versions: [`examples/`](examples). The task lifecycle is `pending → processing → completed | failed`.

## FAQ

**Veo 3.1 모델 id는 무엇인가요?**

`veo3.1-fast-ext`입니다. 위의 라우트 표를 참고하세요.

**과금 방식은 어떻게 되나요?**

**from $0.064 per second** on the APIMart route. 완료된 작업은 `cost` 필드에 정확한 금액을 보고하고, 실패한 작업은 환불됩니다.

**How do I get a key?**

Use the [Get an API key](https://go.apimart.ai/k-bcc772) link — $1 minimum top-up, no plan required.

<!-- related-repos:auto -->
## Related repositories

- **Same model, this language** — the other half of this pair: [veo3-1-prompts-ko](https://github.com/fj2pa3fgmen2yd/veo3-1-prompts-ko)
- **Same model, other languages** — 10 more language builds: English [veo3-1-api](https://github.com/ktq9n4/veo3-1-api) · 中文 [veo3-1-api-cn](https://github.com/wfbjnnovvnj/veo3-1-api-cn) · Deutsch [veo3-1-api-de](https://github.com/mx02yhsjr3t3/veo3-1-api-de) · Español [veo3-1-api-es](https://github.com/bi4gp921v9d/veo3-1-api-es) · Français [veo3-1-api-fr](https://github.com/kd4mi7/veo3-1-api-fr) · Relay/gateway `veo3-1-api-gateway` · Bahasa [veo3-1-api-id](https://github.com/v5m6q1/veo3-1-api-id) · 日本語 [veo3-1-api-ja](https://github.com/gcjb8wo6icy8/veo3-1-api-ja) · Português [veo3-1-api-pt](https://github.com/ea4fa87m8/veo3-1-api-pt) · Русский [veo3-1-api-ru](https://github.com/m67k3hcecku9ef/veo3-1-api-ru)
- **More models in this language** — [midjourney-api-ko](https://github.com/ocvewn2-9ueo/midjourney-api-ko) · [gemini-2-5-flash-image-preview-api-ko](https://github.com/uiyflzgo/gemini-2-5-flash-image-preview-api-ko) · [nano-banana-api-ko](https://github.com/l-2jb09sq/nano-banana-api-ko) · [gemini-3-1-flash-image-preview-api-ko](https://github.com/sgc3a85h2-33l/gemini-3-1-flash-image-preview-api-ko) · [nano-banana-2-api-ko](https://github.com/jjb18r/nano-banana-2-api-ko) · [gemini-3-pro-image-preview-api-ko](https://github.com/a-me4/gemini-3-pro-image-preview-api-ko)
- **Topic hubs** — every build of this model, all languages: [veo3-1](https://github.com/topics/veo3-1) · [veo3-1-api](https://github.com/topics/veo3-1-api)

This repository is one build in a cross-linked set of API guides for the same model family; each language build links back to the others.
<!-- /related-repos:auto -->

## Attributed links (how this repository is measured)

| Purpose | Attributed link | Target |
| --- | --- | --- |
| Model page | <https://go.apimart.ai/k-97128a> | `apimart.ai/ko/model/veo-3-1` |
| Live pricing | <https://go.apimart.ai/k-886aed> | `apimart.ai/ko/pricing` |
| Get an API key | <https://go.apimart.ai/k-bcc772> | `apimart.ai/ko/keys` |

Outbound APIMart links are minted through the promo link API; hand-made tracking parameters are rejected by
`tools/check_links.py` in CI.

## Disclosure

Veo 3.1 is a third-party model served through APIMart; this repository documents how to call it and publishes
model ids and prices, and does not claim official status. Model names, prices and documentation belong to their
respective owners.

## License

MIT — see [LICENSE](LICENSE).
