# 11 · Kontrakt danych (PHP ↔ Python)

Wymiana wyłącznie przez pliki JSON (UTF-8) w katalogu zadania `{storage_root}/fit/jobs/{job_id}/`.
Schematy JSON Schema (draft 2020-12) w `schemas/`: `job.schema.json`, `result.schema.json`,
`fit-model.schema.json`. Obie strony walidują pliki przed użyciem. Wersjonowanie semantyczne `schema_version`:
zmiana niekompatybilna = nowa wersja główna; Python obsługuje bieżącą i poprzednią wersję główną zlecenia.
Nieznana wersja główna `job.json` → `E_SCHEMA_VERSION`; nieznane pola w `job.json` są odrzucane
(`additionalProperties: false`). PHP przyjmuje `result.json` tylko w znanej wersji głównej – inaczej zadanie
`failed` (`E_INTERNAL`) i wpis alertu. Ścieżki w `job.json` są generowane przez serwer i sprawdzane przez Pythona
względem `--storage-root` (`docs/07`, rozdz. 1).

## 1. Zlecenie `job.json` (typ `analyze`)

```json
{
  "schema_version": "1.0.0",
  "job_id": 1842,
  "type": "analyze",
  "paths": {
    "video": "/srv/storage/fit/videos/77/301/source.mp4",
    "out_dir": "/srv/storage/fit/jobs/1842",
    "keyframes_dir": "/srv/storage/fit/keyframes/77/pending-1842"
  },
  "session": {
    "id": 301, "side": "right", "grip": "hoods", "markers_mode": "diy",
    "scale_marker_mm": 200, "cadence_target_rpm": 88, "recording_rpe": 3, "recording_power_w": null,
    "checklist": { "frame_full": true, "light": true, "clothing": true, "markers": true, "scale": false, "warmup": true,
                   "effort": true, "tripod": true }
  },
  "rider": {
    "body": { "height_cm": 182, "mass_kg": 76, "inseam_cm": 86, "arm_span_cm": 184, "foot_length_cm": 27.5,
              "sit_bone_cm": 12.5, "shoulder_width_cm": 40, "foot_arch": "medium",
              "leg_length_diff": "yes", "leg_length_diff_side": "left", "leg_length_diff_mm": 5 },
    "flex": { "posterior_chain": "limited", "ankle": "good", "hip_flexor": "good", "thoracic": "limited",
              "asymmetry": false, "less_flexible": true },
    "complaints": [ { "region": "knee_front", "side": "right", "intensity": 5, "onset": "m60", "type": "pain",
                      "duration_after": "lt2h", "frequency": "always", "off_bike": false, "is_primary": true },
                    { "region": "lower_back", "side": null, "intensity": 3, "onset": "h2", "type": "pressure",
                      "duration_after": "none", "frequency": "half", "off_bike": false, "is_primary": false } ],
    "surgeries": [ { "type": "meniscus", "side": "right", "performed_ym": "2023-04", "restrictions": false } ],
    "injury_recent": false,
    "safety": { "level": "caution", "reasons": ["safety.complaint_5_7", "safety.surgery_past"] },
    "fit_level": "full"
  },
  "bike": {
    "type": "road",
    "geometry": { "stack_mm": 590, "reach_mm": 383, "seat_angle_deg": 73.5, "head_angle_deg": 72.5 },
    "setup": { "id": 55, "saddle_height_mm": 750, "saddle_setback_mm": 62, "saddle_tilt_deg": -1,
               "stem_length_mm": 100, "stem_angle_deg": -6, "spacers_mm": 25, "bar_width_mm": 420,
               "hoods_offset_mm": 0, "crank_length_mm": 172.5 }
  },
  "fit": { "model_version": "fit-model-v1", "profile": "endurance", "accepted": ["saddle_fore_aft"] },
  "limits": { "timeout_s": 900 }
}
```

`calibration.scale_source`: `marker` · `crank` · `wheel` · `body` · `crank_default` · `none`; dodatkowo `scale_sources` (wszystkie oszacowania px/mm), `camera_yaw_deg`, `rectified` (`docs/10`, rozdz. 7). Sekcje `bike.geometry` i pola `bike.setup` są opcjonalne (`null` = nieznane); wymagany jest tylko `bike.type`.

Typy `correct`, `refit`, `simulate` – te same sekcje `rider`, `bike`, `fit` oraz:

| Typ | Dodatkowe pola | Wejście |
|---|---|---|
| `correct` | `base_result`: ścieżka do `result.json` analizy bazowej; `corrections`: `[{ "phase": "bdc", "point": "mtp5", "x_px": 1012, "y_px": 846, "scope": "all_frames" }]`; `tracks`: ścieżka do `tracks.npz` (zapisane przez `analyze` serie punktów) | brak wideo |
| `refit` | `base_result` | brak wideo |
| `simulate` | `base_result`; `changes`: `{ "stem_length_mm": 80, "spacers_mm": 35 }` | brak wideo |

Opcjonalna sekcja `evaluate_change` (typy `analyze` i `refit`) – gdy ustawienie sesji powstało w wyniku zmiany:

```json
"evaluate_change": {
  "change_ids": [912], "params": ["saddle_height_mm"], "single_change": false,
  "before_result": "/srv/storage/fit/results/77/288/result.json",
  "predicted": { "knee_flex_bottom": 38.6 },
  "rides_since_change": 5,
  "ride_feedback": [ { "global": "better", "target_intensity": 4 }, { "global": "much_better", "target_intensity": 3 },
                     { "global": "same", "target_intensity": null } ],
  "target_region": "knee_front",
  "complaints_before": [ { "region": "knee_front", "intensity": 5, "is_primary": true } ],
  "complaints_after": [ { "region": "knee_front", "intensity": 3, "is_primary": true } ]
}
```

`ride_feedback[].global`: `much_better` · `better` · `same` · `worse` · `much_worse` (`docs/09`, rozdz. 9).
`rider.safety.level` i `rider.fit_level` wylicza PHP (`docs/04`, rozdz. 5.1 i 8); Python stosuje ich skutki
(`docs/09`, rozdz. 5.1–5.2) i nie wylicza ich ponownie.

Wynik zawiera wtedy sekcję `change_evaluation`: `{ "verdict": "effective", "comparable": true, "criteria": { "executed_as_planned": "yes", "in_range": "yes", "feels_better": "yes", "comparable": "yes", "single_change": "no" }, "notes_keys": ["eval.multi_change"] }` (reguły: `docs/09`, rozdz. 9).

`scope` korekty: `all_frames` (domyślnie) – przesunięcie punktu wyrażone w lokalnym układzie segmentu nadrzędnego
(np. kolano względem uda) stosowane do całej trajektorii, potem ponowne obliczenie pomiarów, zaleceń i wariantów;
`keyframe` – korekta tylko punktu w danym kadrze (wpływa na rysunek i warianty tej fazy, nie na statystyki cykli).

## 2. Wynik `result.json`

```json
{
  "schema_version": "1.0.0",
  "job_id": 1842, "type": "analyze", "status": "ok_with_warnings",
  "error": null,
  "algorithm": { "version": "0.3.0", "pose_model": "mediapipe_full", "pose_model_version": "0.10.x",
                 "pipeline_config_hash": "sha256:…", "processing_ms": { "total": 412000, "detect": 301000 } },
  "video": { "duration_s": 52.4, "fps": 59.94, "width": 1920, "height": 1080, "rotation": 0, "codec": "h264",
             "fragment": { "start_s": 6.0, "end_s": 50.0 } },
  "calibration": { "scale_px_per_mm": 1.94, "scale_source": "marker", "camera_tilt_deg": 0.8,
                   "tilt_source": "floor_line", "direction": "right", "bb_px": [930, 812], "bb_fit_rmse_mm": 4.1 },
  "quality": {
    "overall": "warn", "frames_ok_pct": 91.2, "cycles_total": 46, "cycles_valid": 41,
    "points": { "hip": { "mean_conf": 0.97, "marker_visible_pct": 99.1 }, "mtp5": { "mean_conf": 0.71, "marker_visible_pct": 62.0 } },
    "warnings": [ { "code": "W_MARKER_OCCLUDED", "point": "mtp5" }, { "code": "W_CHECKLIST", "item": "scale" } ]
  },
  "cycles": [ { "n": 1, "valid": false, "reject_reason": "incomplete", "t_start_s": 6.0, "t_end_s": 6.6, "cadence_rpm": null },
              { "n": 2, "valid": true, "reject_reason": null, "t_start_s": 6.6, "t_end_s": 7.29, "cadence_rpm": 87 } ],
  "keyframes": [
    { "phase": "bdc", "frame_index": 1476, "time_s": 24.6, "image_file": "bdc.jpg", "width": 1920, "height": 1080,
      "points": { "hip": { "x_px": 702, "y_px": 402, "x_mm": -200.4, "y_mm": 765.2, "source": "marker", "confidence": 0.97 },
                  "mtp5": { "x_px": 1012, "y_px": 846, "x_mm": 41.0, "y_mm": -18.3, "source": "model", "confidence": 0.64,
                            "note": "marker_occluded" } } }
  ],
  "metrics": {
    "knee_flex_bottom": { "unit": "deg", "quality": "good", "n_cycles": 41, "validation_status": "validated",
                          "values": { "med": { "value": 38.3, "sd": 1.4 } } },
    "knee_angle": { "unit": "deg", "quality": "good", "n_cycles": 41,
                    "values": { "max": { "value": 141.7, "sd": 1.4 }, "min": { "value": 71.4, "sd": 0.8 }, "rom": { "value": 70.3, "sd": 1.6 } },
                    "series": { "mean": [71.9, "… 101 wartości"], "sd": [0.8, "…"] } }
  },
  "body_model": { "segments_mm": { "thigh": 421, "shank": 478, "foot": 151, "torso": 579, "upper_arm": 304, "forearm": 276 },
                  "offsets_mm": { "hip_from_saddle": [13, 46], "hand_from_bar": [130, -35] }, "approximate": false,
                  "bike_from_video": { "crank_length_mm": 172.0, "saddle_height_mm": 748, "saddle_bar_drop_mm": 64,
                                       "seat_angle_deg": 73.1 } },
  "fit": {
    "model_version": "fit-model-v1", "profile": "endurance", "bike_type": "road",
    "fit_level": "full", "safety_level": "caution",
    "modifiers": [ { "code": "M_LESS_FLEX", "text_key": "mod.less_flex" }, { "code": "M_LOW_BACK", "text_key": "mod.low_back" } ],
    "regulations": {
      "saddle_height": { "status": "ok", "direction": null, "value": 38.3, "unit": "deg", "rule_status": "experimental",
                         "range": { "green": [33, 40], "yellow": [25, 45], "target": 36.5, "disputed": [] },
                         "method": "dynamic_2d_lateral_markers", "evidence": "science", "flags": [] },
      "bar_reach": { "status": "near", "direction": "above", "value": 166.0, "unit": "deg", "rule_status": "validated",
                     "range": { "green": [148, 165], "yellow": [140, 170], "target": 156, "disputed": [] },
                     "method": "dynamic_2d_lateral_markers", "evidence": "practice", "flags": [] }
    },
    "recommendations": [
      { "order": 1, "regulation": "bar_reach", "action_key": "rec.bar_reach.shorter", "param": "stem_length_mm",
        "amount_mm": 5, "amount_total_mm": 20, "variant_id": "stem_length_mm:-10",
        "confidence": "medium", "confidence_reasons": ["conf.near_boundary"],
        "effort": { "minutes": 30, "tools": ["tool.stem", "tool.hex_4_5", "tool.torque"], "difficulty": "medium",
                    "shop_recommended": true },
        "text": { "title": "Skróć mostek o ok. 5 mm", "why": "…", "how": "…", "caution": "Małymi krokami…" },
        "rule_status": "validated",
        "side_effects": { "torso_angle": 0.6, "knee_over_pedal": 0.0 },
        "rationale": { "measurement": { "metric": "elbow_angle", "value": 166.0, "sd": 1.9, "quality": "good",
                                        "validation_status": "validated" },
                       "base_range": { "green": [148, 165], "source_ids": ["…"], "evidence": "practice" },
                       "layers": [ { "code": "M_LOW_BACK", "effect": "block:bar_height.lower" } ],
                       "decision": "above_green_by_1.0", "order_reason": "plan.step1", "model_version": "fit-model-v1" },
        "blocked_by": null }
    ],
    "next_step": 0,
    "plan": [ { "step": 1, "recommendation": 0, "predicted": { "elbow_angle": 160.4 }, "tentative": false },
              { "step": 2, "regulation": "bar_height", "param": "spacers_mm", "delta": 10,
                "predicted": { "torso_angle": 41.2 }, "tentative": true } ],
    "tips": [ { "key": "tip.saddle_width", "params": { "from_mm": 145, "to_mm": 155 } },
              { "key": "tip.crank_length", "params": { "variant_id": "crank_length_mm:-2.5" } } ],
    "signals": { "pelvis_rock": false },
    "missing_data": [ { "field": "stem_length_mm", "purpose_key": "missing.visualize_and_fit" } ]
  },
  "variants": [
    { "id": "stem_length_mm:-10", "param": "stem_length_mm", "value": 90, "delta": -10, "approximate": false,
      "geometry": { "bar_center_mm": [473, 637], "saddle_to_bar_mm": 532, "saddle_bar_drop_mm": 61 },
      "prediction_validated": false,
      "metrics": { "elbow_angle": 160.4, "torso_angle": 40.6, "knee_flex_bottom": 38.3, "knee_over_pedal": -33.7, "hands_load_share": 31.2 },
      "statuses": { "bar_reach": "ok", "bar_height": "ok", "saddle_height": "ok", "saddle_fore_aft": "ok" },
      "loads": { "hands": "medium", "neck": "medium", "low_back": "low" }, "flags": [],
      "skeleton": { "p90": { "hip": [702, 402], "knee": [1001, 655], "ankle": [905, 1085], "mtp5": [1046, 1110],
                             "heel": [858, 1120], "shoulder": [1123, 75], "elbow": [1240, 338], "wrist": [1441, 496] } } }
  ]
}
```

Przy `safety_level = stop`: `recommendations = []`, `next_step = null`, regulacje i pomiary bez zmian. Pomiary
w `metrics` obejmują także `knee_flex_top` (`docs/08`). Ostrzeżenie `W_HIGH_EFFORT` – `docs/07`, rozdz. 9.

Pliki towarzyszące w `out_dir`: `tracks.npz` (serie punktów dla `correct`), kadry `tdc.jpg`, `p90.jpg`, `bdc.jpg`,
`p270.jpg` w `keyframes_dir`, `worker.log` (bez danych osobowych).

Sekcje `fit` i `variants` są opcjonalne w schemacie: w etapach M1–M3 (`docs/17`) wynik zawiera tylko pomiar;
`variants` i `plan` pojawiają się w M5.

`status`: `ok` · `ok_with_warnings` · `error`. Przy `error` wypełnione `error: { "code", "message_key", "details" }`
(`details` bez danych osobowych), pozostałe sekcje mogą być puste.

## 3. Mapowanie na bazę (worker PHP)

| Sekcja | Tabela |
|---|---|
| `algorithm`, `video`, `calibration`, `quality` (nagłówek), `fit.fit_level`, `fit.safety_level` | `fit_analysis` |
| `keyframes[]` | `fit_keyframe` + `fit_point` |
| `metrics` (bez `series`) | `fit_metric` |
| `cycles[]` | `fit_cycle` |
| `fit.regulations`, `fit.recommendations` | `fit_regulation`, `fit_recommendation` |
| `fit.modifiers` | `fit_analysis_modifier` |
| `variants[]` | `fit_variant` (metryki i statusy w kolumnach JSON, szkielety w JSON) |
| cały plik | `results/{user}/{analysis}/result.json` (serie do wykresów, szczegóły) |

## 4. Kody punktów

`hip`, `knee`, `ankle`, `heel`, `mtp5`, `shoulder`, `elbow`, `wrist`, `head` (orientacyjnie), `bb`, `pedal`,
`scale_a`, `scale_b`.

## 5. Kody faz

`tdc` (0°, korba w górze), `p90` (90°, z przodu), `bdc` (180°, na dole), `p270` (270°, z tyłu).

## 6. Kody pomiarów, regulacji, modyfikatorów

Pomiary: `docs/08`, rozdz. 3. Regulacje i modyfikatory: `docs/09`, rozdz. 1 i 5.

## 7. Kody błędów i komunikaty

| Kod | Ponowić? | Komunikat dla użytkownika (PL) |
|---|---|---|
| `E_VIDEO_FORMAT` | nie | Nieobsługiwany format pliku. Prześlij nagranie MP4 lub MOV. |
| `E_VIDEO_TOO_LARGE` | nie | Plik jest za duży (limit {limit}). Skróć nagranie do ok. 60 s. |
| `E_VIDEO_CODEC` | nie | Nie możemy odczytać tego nagrania. W telefonie wybierz format „Najbardziej zgodny” (H.264) i nagraj ponownie. |
| `E_VIDEO_TOO_SHORT` | nie | Nagranie jest za krótkie. Potrzebujemy co najmniej 30 s jazdy w stałej kadencji. |
| `E_VIDEO_FPS_LOW` | nie | Nagranie ma za mało klatek na sekundę. Ustaw w aparacie 60 kl./s (minimum 30). |
| `E_VIDEO_RES_LOW` | nie | Rozdzielczość jest za niska. Nagraj w 1080p (minimum 720p). |
| `E_POSE_COVERAGE` | nie | Nie widać całej sylwetki w {pct}% klatek. Najczęściej: nagranie pod światło lub ktoś/coś zasłania. |
| `E_TRACKING_LOST` | nie | Zgubiliśmy punkty ciała w zbyt wielu klatkach. Sprawdź, czy cała noga i ręka są w kadrze. |
| `E_CYCLES_TOO_FEW` | nie | Za mało równych obrotów korby. Nagraj min. 30 s w stałej kadencji. |
| (usunięty) `E_SCALE_INVALID` | – | nieczytelny znacznik nie jest już błędem – ostrzeżenie `W_SCALE_MARKER_UNREADABLE` i inne źródło skali (ADR-28) |
| `E_TIMEOUT` | tak | Analiza trwała za długo. Spróbujemy ponownie automatycznie. |
| `E_MEMORY` | tak | Zabrakło zasobów serwera. Spróbujemy ponownie automatycznie. |
| `E_JOB_INVALID` | nie | Wystąpił błąd analizy (kod zdarzenia {id}). Zgłosiliśmy go – nie musisz nic robić. |
| `E_SCHEMA_VERSION` | nie | jw. |
| `E_INTERNAL` | tak (maks. 3) | Wystąpił błąd analizy. Spróbujemy ponownie; jeśli się powtórzy – skontaktuj się z nami (kod zdarzenia {id}). |

Ostrzeżenia (`W_*`) – lista w `docs/07`, rozdz. 9; teksty w pliku tłumaczeń `templates/i18n/pl.php`.
