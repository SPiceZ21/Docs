# spz-fpscap

> 60 FPS fairness cap.

![version](https://img.shields.io/badge/version-1.0.0-ff6200) ![SPiceZ-Core](https://img.shields.io/badge/SPiceZ--Core-beta-18181b)

## Overview

Detects players running above 60 FPS and blocks them until they cap it, so physics behaves the same for everyone.

## Features

- FPS detection
- Blocking prompt until capped

## Requirements

None — runs on its own.

## Installation

Included in the [txAdmin recipe](https://github.com/SPiceZ21/spz-txrecipe). Manual install:

1. Download `spz-fpscap` from GitHub (latest release, or the repository) into your `resources` folder.
2. Add to `server.cfg`:

```cfg
ensure spz-fpscap
```

## Configuration

- `config.lua`

See the file for every option.

---

Source: [https://github.com/SPiceZ21/spz-fpscap](https://github.com/SPiceZ21/spz-fpscap)
