<script setup>
import P5 from 'p5'
import { onBeforeUnmount, onMounted, ref } from 'vue'

const hostRef = ref(null)
const labelLayerRef = ref(null)

let sketch = null
let resizeObserver = null
let cleanupPointerEvents = null
let cleanupLabelModeQuery = null

const pointCount = 225
const maxConnectionsPerDot = 8
const maxLongLinkDistance = 3
const connectionColors = [
  { r: 45, g: 212, b: 191 },
  { r: 56, g: 189, b: 248 },
  { r: 129, g: 140, b: 248 },
  { r: 168, g: 85, b: 247 },
  { r: 244, g: 114, b: 182 },
  { r: 251, g: 146, b: 60 },
  { r: 250, g: 204, b: 21 },
  { r: 52, g: 211, b: 153 },
]
const scanLabels = [
  'Slack',
  'Gmail',
  'Notion',
  'Drive',
  'Docs',
  'Calendar',
  'CRM',
  'Files',
  'Tasks',
  'Clients',
  'Projects',
  'Receipts',
  'Inbox',
  'Meetings',
  'Decisions',
  'Notes',
]

const seededRandom = (seed) => {
  const value = Math.sin(seed * 12.9898) * 43758.5453

  return value - Math.floor(value)
}

const easeInOutCubic = (value) =>
  value < 0.5 ? 4 * value * value * value : 1 - Math.pow(-2 * value + 2, 3) / 2

const clamp01 = (value) => Math.min(Math.max(value, 0), 1)

const createSpherePoints = () => {
  const goldenAngle = Math.PI * (3 - Math.sqrt(5))

  return Array.from({ length: pointCount }, (_, index) => {
    const y = 1 - (index / (pointCount - 1)) * 2
    const radius = Math.sqrt(1 - y * y)
    const theta = goldenAngle * index
    const hubScore = seededRandom(index * 31 + 503)
    const spherePulse = 0.78 + seededRandom(index + 47) * 0.18

    return {
      color: connectionColors[Math.floor(seededRandom(index * 149 + 61) * connectionColors.length)],
      hubScore,
      sphere: {
        x: Math.cos(theta) * radius * spherePulse,
        y: y * spherePulse,
        z: Math.sin(theta) * radius * spherePulse,
      },
    }
  })
}

const createLinks = (points) => {
  const links = []
  const usedLinks = new Set()
  const connectionCounts = points.map(() => 0)
  const addLink = (fromIndex, toIndex, strength = 1) => {
    const from = Math.min(fromIndex, toIndex)
    const to = Math.max(fromIndex, toIndex)
    const key = `${from}-${to}`

    if (from === to || usedLinks.has(key)) return false
    if (connectionCounts[from] >= maxConnectionsPerDot || connectionCounts[to] >= maxConnectionsPerDot) return false

    usedLinks.add(key)
    connectionCounts[from] += 1
    connectionCounts[to] += 1
    links.push({
      color: connectionColors[Math.floor(seededRandom(from * 173 + to * 257) * connectionColors.length)],
      from,
      strength,
      to,
    })

    return true
  }

  points.forEach((point, pointIndex) => {
    const localNeighborCount = 1 + Math.floor(seededRandom(pointIndex + 157) * 2)
    const nearest = points
      .map((candidate, candidateIndex) => {
        if (candidateIndex === pointIndex) return null

        return {
          index: candidateIndex,
          distance:
            Math.pow(point.sphere.x - candidate.sphere.x, 2) +
            Math.pow(point.sphere.y - candidate.sphere.y, 2) +
            Math.pow(point.sphere.z - candidate.sphere.z, 2),
        }
      })
      .filter(Boolean)
      .sort((a, b) => a.distance - b.distance)
      .slice(0, localNeighborCount)

    nearest.forEach((candidate) => {
      addLink(pointIndex, candidate.index, 0.82)
    })

    const randomizedCrossLinks = 1 + Math.floor(seededRandom(pointIndex + 211) * 3)
    const longLinkCount =
      randomizedCrossLinks + (point.hubScore > 0.82 ? 3 : point.hubScore > 0.68 ? 2 : 0)

    for (let linkIndex = 0; linkIndex < longLinkCount; linkIndex += 1) {
      for (let attempt = 0; attempt < 20; attempt += 1) {
        const candidateIndex = Math.floor(
          seededRandom(pointIndex * 419 + linkIndex * 313 + attempt * 137) * points.length
        )
        const candidate = points[candidateIndex]
        const distance =
          Math.pow(point.sphere.x - candidate.sphere.x, 2) +
          Math.pow(point.sphere.y - candidate.sphere.y, 2) +
          Math.pow(point.sphere.z - candidate.sphere.z, 2)

        if (candidateIndex === pointIndex || distance < 0.48 || distance > maxLongLinkDistance) continue

        if (addLink(pointIndex, candidateIndex, point.hubScore > 0.68 ? 0.5 : 0.38)) break
      }
    }
  })

  points.forEach((_point, pointIndex) => {
    if (connectionCounts[pointIndex] > 0) return

    const nearest = points
      .map((candidate, candidateIndex) => {
        if (candidateIndex === pointIndex) return null

        return {
          index: candidateIndex,
          distance:
            Math.pow(points[pointIndex].sphere.x - candidate.sphere.x, 2) +
            Math.pow(points[pointIndex].sphere.y - candidate.sphere.y, 2) +
            Math.pow(points[pointIndex].sphere.z - candidate.sphere.z, 2),
        }
      })
      .filter(Boolean)
      .sort((a, b) => a.distance - b.distance)

    nearest.some((candidate) => addLink(pointIndex, candidate.index, 0.82))
  })

  const orderedLinks = [...links].sort(
    (a, b) =>
      seededRandom(a.from * 97 + a.to * 193) -
      seededRandom(b.from * 97 + b.to * 193)
  )
  const stageSizes = [1, 1, 2, 5, 10]
  const stageStarts = [0, 0.12, 0.24, 0.36, 0.48]
  const stageSpans = [0, 0, 0.03, 0.045, 0.06]
  let cursor = 0

  stageSizes.forEach((stageSize, stageIndex) => {
    for (let index = 0; index < stageSize && cursor < orderedLinks.length; index += 1) {
      const stageProgress = stageSize > 1 ? index / (stageSize - 1) : 0

      orderedLinks[cursor].revealDelay =
        stageStarts[stageIndex] + stageProgress * stageSpans[stageIndex]
      cursor += 1
    }
  })

  const remainingCount = orderedLinks.length - cursor

  for (let index = 0; cursor < orderedLinks.length; index += 1) {
    const progress = remainingCount > 1 ? index / (remainingCount - 1) : 0

    orderedLinks[cursor].revealDelay = 0.6 + progress * 0.18
    cursor += 1
  }

  return links
}

const points = createSpherePoints()
const links = createLinks(points)
const connectionCounts = links.reduce((counts, link) => {
  counts[link.from] += 1
  counts[link.to] += 1

  return counts
}, points.map(() => 0))
const maxConnectionCount = Math.max(...connectionCounts)

onMounted(() => {
  sketch = new P5((p) => {
    let activeLabels = []
    let hoverPoint = { active: false, x: 0, y: 0 }
    let isAnimatedLabelMode = false
    let labelCursor = 0
    let lastAnimatedLabelStartedAt = -Infinity
    let lastLabelPickedAt = -Infinity

    const maxAnimatedLabels = 2
    const hoverPickInterval = 90
    const animatedLabelStartDelay = 700
    const animatedLabelStagger = 1150
    const animatedLabelFadeInDuration = 320
    const animatedLabelHoldDuration = 1500
    const animatedLabelFadeOutDuration = 520
    const animatedLabelDuration =
      animatedLabelFadeInDuration + animatedLabelHoldDuration + animatedLabelFadeOutDuration
    const labelFadeInEase = 0.18
    const labelFadeOutEase = 0.12

    const clearLabels = () => {
      activeLabels.forEach((label) => {
        label.element?.remove()
      })
      activeLabels = []
    }

    const createLabelElement = (text) => {
      const element = document.createElement('div')

      element.textContent = text
      Object.assign(element.style, {
        color: '#fff',
        fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
        fontWeight: '500',
        left: '0',
        lineHeight: '1',
        opacity: '0',
        pointerEvents: 'none',
        position: 'absolute',
        top: '0',
        transform: 'translate3d(-999px, -999px, 0)',
        transformOrigin: 'center bottom',
        whiteSpace: 'nowrap',
        willChange: 'transform, opacity',
      })

      labelLayerRef.value?.appendChild(element)

      return element
    }

    const setupPointerEvents = () => {
      const element = hostRef.value

      if (!element) return

      const updateHoverPoint = (event) => {
        const rect = element.getBoundingClientRect()

        hoverPoint = {
          active: true,
          x: event.clientX - rect.left,
          y: event.clientY - rect.top,
        }
      }
      const clearHoverPoint = () => {
        hoverPoint = { ...hoverPoint, active: false }
      }

      element.addEventListener('pointerenter', updateHoverPoint)
      element.addEventListener('pointermove', updateHoverPoint)
      element.addEventListener('pointerleave', clearHoverPoint)

      cleanupPointerEvents = () => {
        element.removeEventListener('pointerenter', updateHoverPoint)
        element.removeEventListener('pointermove', updateHoverPoint)
        element.removeEventListener('pointerleave', clearHoverPoint)
      }
    }

    const setupLabelModeQuery = () => {
      const query = window.matchMedia('(max-width: 767px)')
      const setLabelMode = (shouldAnimate) => {
        if (isAnimatedLabelMode === shouldAnimate) return

        isAnimatedLabelMode = shouldAnimate
        hoverPoint = { active: false, x: 0, y: 0 }
        lastAnimatedLabelStartedAt = -Infinity
        lastLabelPickedAt = -Infinity
        clearLabels()
      }
      const handleChange = (event) => {
        setLabelMode(event.matches)
      }

      isAnimatedLabelMode = query.matches
      query.addEventListener('change', handleChange)

      cleanupLabelModeQuery = () => {
        query.removeEventListener('change', handleChange)
      }
    }

    const rotatePoint = (point, time) => {
      const tilt = -0.42
      const roll = -0.12
      const yaw = time
      const cosY = Math.cos(yaw)
      const sinY = Math.sin(yaw)
      const cosX = Math.cos(tilt)
      const sinX = Math.sin(tilt)
      const cosZ = Math.cos(roll)
      const sinZ = Math.sin(roll)
      const x1 = point.x * cosY + point.z * sinY
      const z1 = -point.x * sinY + point.z * cosY
      const y2 = point.y * cosX - z1 * sinX
      const z2 = point.y * sinX + z1 * cosX

      return {
        x: x1 * cosZ - y2 * sinZ,
        y: x1 * sinZ + y2 * cosZ,
        z: z2,
      }
    }

    const projectPoint = (point, size, time) => {
      const center = size * 0.5
      const radius = size * 0.45
      const cameraDistance = radius * 3.4
      const rotated = rotatePoint(point.sphere, time)
      const z = rotated.z * radius
      const perspective = cameraDistance / (cameraDistance - z)

      return {
        color: point.color,
        x: center + rotated.x * radius * perspective,
        y: center + rotated.y * radius * perspective,
        z: rotated.z,
      }
    }

    const getQuadraticPoint = (from, control, to, progress) => {
      const inverse = 1 - progress

      return {
        x: inverse * inverse * from.x + 2 * inverse * progress * control.x + progress * progress * to.x,
        y: inverse * inverse * from.y + 2 * inverse * progress * control.y + progress * progress * to.y,
      }
    }

    const getFogOpacity = (z, minimum = 0.14) => {
      const normalizedDepth = clamp01((z + 0.85) / 1.7)

      return minimum + Math.pow(normalizedDepth, 1.45) * (1 - minimum)
    }

    const getEndpointOpacity = (index) => {
      if (!maxConnectionCount) return 0.08

      return 0.08 + Math.pow(connectionCounts[index] / maxConnectionCount, 1.35) * 0.32
    }

    const getNextLabelText = () => {
      const label = scanLabels[labelCursor % scanLabels.length]

      labelCursor += 1

      return label
    }

    const getAnimatedLabelOpacity = (label) => {
      const elapsed = p.millis() - label.startedAt

      if (elapsed < animatedLabelFadeInDuration) {
        return easeInOutCubic(clamp01(elapsed / animatedLabelFadeInDuration))
      }

      if (elapsed < animatedLabelFadeInDuration + animatedLabelHoldDuration) {
        return 1
      }

      return 1 - easeInOutCubic(
        clamp01(
          (elapsed - animatedLabelFadeInDuration - animatedLabelHoldDuration) /
            animatedLabelFadeOutDuration
        )
      )
    }

    const getHoverStrength = (point, size) => {
      if (!hoverPoint.active || point.z <= 0) return 0

      const radius = size * 0.28
      const distance = Math.hypot(point.x - hoverPoint.x, point.y - hoverPoint.y)
      const progress = clamp01(distance / radius)

      return Math.pow(1 - easeInOutCubic(progress), 1.15)
    }

    const pickHoverCandidate = (candidates) => {
      const totalWeight = candidates.reduce(
        (total, candidate) => total + candidate.strength,
        0
      )
      let roll = seededRandom(p.millis() * 0.017 + labelCursor * 101) * totalWeight

      return candidates.find((candidate) => {
        roll -= candidate.strength

        return roll <= 0
      }) || candidates[0]
    }

    const updateAnimatedLabels = (projectedPoints, size) => {
      const now = p.millis()

      activeLabels = activeLabels.filter((label) => {
        const isVisible = now - label.startedAt < animatedLabelDuration

        if (!isVisible) {
          label.element?.remove()
        }

        return isVisible
      })

      if (
        labelLayerRef.value &&
        now > animatedLabelStartDelay &&
        activeLabels.length < maxAnimatedLabels &&
        now - lastAnimatedLabelStartedAt > animatedLabelStagger
      ) {
        const activeIndexes = new Set(activeLabels.map((label) => label.index))
        const candidates = projectedPoints
          .map((point, index) => ({ ...point, index }))
          .filter((point) => {
            return (
              !activeIndexes.has(point.index) &&
              point.z > -0.18 &&
              point.x > size * 0.1 &&
              point.x < size * 0.9 &&
              point.y > size * 0.08 &&
              point.y < size * 0.9
            )
          })

        if (candidates.length) {
          const candidate =
            candidates[
              Math.floor(seededRandom(now * 0.013 + labelCursor * 89) * candidates.length)
            ]

          activeLabels.push({
            element: createLabelElement(getNextLabelText()),
            index: candidate.index,
            currentOpacity: 0,
            currentScale: 1,
            startedAt: now,
          })
          lastAnimatedLabelStartedAt = now
        }
      }

      activeLabels.forEach((label) => {
        const point = projectedPoints[label.index]

        label.currentOpacity = point
          ? getAnimatedLabelOpacity(label) * getFogOpacity(point.z, 0.42)
          : 0
      })
    }

    const updateHoverLabels = (projectedPoints, size) => {
      const now = p.millis()
      const candidates = projectedPoints
        .map((point, index) => ({ ...point, index }))
        .map((point) => ({
          ...point,
          strength: getHoverStrength(point, size),
        }))
        .filter((point) => {
          return point.strength > 0.08
        })
      const candidateByIndex = new Map(
        candidates.map((candidate) => [candidate.index, candidate])
      )

      activeLabels.forEach((label) => {
        const candidate = candidateByIndex.get(label.index)

        label.targetOpacity = candidate
          ? candidate.strength * getFogOpacity(candidate.z, 0.42)
          : 0
        label.targetScale = candidate
          ? 0.5 + candidate.strength * 0.5
          : 0.5
      })

      activeLabels = activeLabels.filter((label) => {
        const ease = label.targetOpacity > label.currentOpacity
          ? labelFadeInEase
          : labelFadeOutEase

        label.currentOpacity += (label.targetOpacity - label.currentOpacity) * ease
        label.currentScale += ((label.targetScale ?? 1) - (label.currentScale ?? 1)) * ease

        if (label.currentOpacity > 0.01 || label.targetOpacity > 0.01) return true

        label.element?.remove()

        return false
      })

      if (!labelLayerRef.value || !hoverPoint.active || !candidates.length) return

      if (now - lastLabelPickedAt < hoverPickInterval) return

      const activeIndexes = new Set(activeLabels.map((label) => label.index))
      const availableCandidates = candidates.filter((candidate) => {
        return !activeIndexes.has(candidate.index)
      })

      if (!availableCandidates.length) return

      const revealCount = Math.min(3, availableCandidates.length)

      for (let index = 0; index < revealCount; index += 1) {
        const candidate = pickHoverCandidate(availableCandidates)
        const candidateIndex = availableCandidates.indexOf(candidate)

        availableCandidates.splice(candidateIndex, 1)
        activeLabels.push({
          element: createLabelElement(getNextLabelText()),
          index: candidate.index,
          createdAt: now,
          currentOpacity: 0,
          currentScale: 0.5,
          targetOpacity: candidate.strength * getFogOpacity(candidate.z, 0.42),
          targetScale: 0.5 + candidate.strength * 0.5,
        })
      }
      lastLabelPickedAt = now
    }

    const updateLabelElement = (label, point, size) => {
      if (!label.element) return

      const fontSize = Math.max(10, Math.min(13, size * 0.026))
      const paddingX = fontSize * 0.7
      const paddingY = fontSize * 0.38
      const labelHeight = fontSize + paddingY * 2

      label.element.style.backgroundColor = `rgb(${point.color.r}, ${point.color.g}, ${point.color.b})`
      label.element.style.borderRadius = `${labelHeight * 0.5}px`
      label.element.style.fontSize = `${fontSize}px`
      label.element.style.opacity = `${label.currentOpacity}`
      label.element.style.padding = `${paddingY}px ${paddingX}px`
      label.element.style.transform = `translate3d(${point.x}px, ${point.y - fontSize * 1.15}px, 0) translate(-50%, -100%) scale(${label.currentScale ?? 1})`
    }

    const drawScanNetwork = () => {
      const size = Math.min(p.width, p.height)
      const cycleAlpha = 1
      const linkProgress = clamp01(p.millis() / 5500)
      const time = p.millis() * 0.00014
      const projectedPoints = points.map((point) =>
        projectPoint(point, size, time)
      )
      const visibleConnectionCounts = points.map(() => 0)

      p.clear()
      if (isAnimatedLabelMode) {
        updateAnimatedLabels(projectedPoints, size)
      } else {
        updateHoverLabels(projectedPoints, size)
      }

      links.forEach((link) => {
        const from = projectedPoints[link.from]
        const to = projectedPoints[link.to]
        const distance = Math.hypot(to.x - from.x, to.y - from.y)
        const trimDuration = 0.08 + clamp01(distance / (size * 0.72)) * 0.14
        const reveal = easeInOutCubic(clamp01((linkProgress - link.revealDelay) / trimDuration))
        const depth = getFogOpacity((from.z + to.z) * 0.5, 0.1)

        if (reveal <= 0.01) return

        visibleConnectionCounts[link.from] += reveal
        visibleConnectionCounts[link.to] += reveal

        const center = size * 0.5
        const midpointX = (from.x + to.x) * 0.5
        const midpointY = (from.y + to.y) * 0.5
        const outwardX = midpointX - center
        const outwardY = midpointY - center
        const outwardDistance = Math.hypot(outwardX, outwardY) || 1
        const archHeight = Math.min(size * 0.204, Math.pow(distance / size, 1.12) * size * 0.552)
        const control = {
          x: midpointX + (outwardX / outwardDistance) * archHeight,
          y: midpointY + (outwardY / outwardDistance) * archHeight,
        }
        const trimProgress = Math.min(reveal, 1)
        const segmentCount = Math.max(8, Math.ceil(24 * trimProgress))
        const context = p.drawingContext
        const fromOpacity = getEndpointOpacity(link.from)
        const toOpacity = getEndpointOpacity(link.to)

        context.save()
        for (let segment = 1; segment <= segmentCount; segment += 1) {
          const previousProgress = ((segment - 1) / segmentCount) * trimProgress
          const currentProgress = (segment / segmentCount) * trimProgress
          const previousPoint = getQuadraticPoint(
            from,
            control,
            to,
            previousProgress
          )
          const currentPoint = getQuadraticPoint(
            from,
            control,
            to,
            currentProgress
          )
          const progress = (previousProgress + currentProgress) * 0.5
          const endpointOpacity = Math.max(
            fromOpacity * Math.pow(1 - progress, 1.65),
            toOpacity * Math.pow(progress, 1.65),
            0.025
          )

          context.beginPath()
          context.moveTo(previousPoint.x, previousPoint.y)
          context.lineTo(currentPoint.x, currentPoint.y)
          context.globalAlpha = cycleAlpha * depth * endpointOpacity
          context.strokeStyle = `rgb(${link.color.r}, ${link.color.g}, ${link.color.b})`
          context.lineWidth = link.strength > 0.7 ? 0.55 : 0.42
          context.lineCap = 'round'
          context.stroke()
        }
        context.restore()
      })

      projectedPoints.forEach((point, index) => {
        const depth = getFogOpacity(point.z, 0.18)
        const connectionScale = maxConnectionCount
          ? Math.pow(visibleConnectionCounts[index] / maxConnectionCount, 1.85)
          : 0
        const dotSize = 2.15 + connectionScale * 1.6

        p.noStroke()
        p.fill(point.color.r, point.color.g, point.color.b, 230 * depth * cycleAlpha)
        p.circle(point.x, point.y, dotSize)
      })

      activeLabels
        .map((label) => ({
          ...label,
          point: projectedPoints[label.index],
        }))
        .filter(({ point }) => point)
        .sort((a, b) => a.point.z - b.point.z)
        .forEach((label) => {
          updateLabelElement(label, label.point, size)
        })
    }

    p.setup = () => {
      const size = hostRef.value?.clientWidth || 320
      const canvas = p.createCanvas(size, size)

      canvas.parent(hostRef.value)
      canvas.elt.style.background = 'transparent'
      p.pixelDensity(Math.min(window.devicePixelRatio || 1, 3))
      p.smooth()
      setupPointerEvents()
      setupLabelModeQuery()
    }

    p.draw = drawScanNetwork
  })

  resizeObserver = new ResizeObserver(([entry]) => {
    if (!entry || !sketch) return

    const size = Math.round(entry.contentRect.width)

    sketch.resizeCanvas(size, size)
  })

  if (hostRef.value) {
    resizeObserver.observe(hostRef.value)
  }
})

onBeforeUnmount(() => {
  cleanupPointerEvents?.()
  cleanupPointerEvents = null
  cleanupLabelModeQuery?.()
  cleanupLabelModeQuery = null
  labelLayerRef.value?.replaceChildren()
  resizeObserver?.disconnect()
  sketch?.remove()
})
</script>

<template>
  <div class="aspect-square w-full overflow-hidden rounded-3xl p-8" aria-hidden="true">
    <div class="relative size-full">
      <div
        ref="hostRef"
        class="absolute inset-0 overflow-hidden [&_canvas]:block [&_canvas]:size-full"
      />
      <div ref="labelLayerRef" class="pointer-events-none absolute inset-0 z-10" />
    </div>
  </div>
</template>
