<script>
  import { onMount } from "svelte"

  export let canvasSizePx
  export let strokeWidth
  export let targetType
  export let arrowRadius
  export let arrows
  export let addArrow

  const maxRadius = canvasSizePx / 2

  let targetScoreRange

  const getTarget = () => {
    if (targetType === "nfaa") {
      const spacing = maxRadius / 6
      targetScoreRange = NFAA_TARGET(spacing)
    } else {
      const spacing = maxRadius / 10
      targetScoreRange = OLYMPIC_TARGET(spacing)
    }
  }

  const NFAA_TARGET = (spacing) => {
    return [
      {
        arrowColor: 'black',
        color: 'white',
        maxRadius: (maxRadius - spacing * 5),
        points: 5,
        strokeColor: '#233b5e',
        strokeWidth: 0.5
      },
      {
        arrowColor: 'black',
        color: 'white',
        maxRadius: (maxRadius - spacing * 4),
        points: 5,
        strokeColor: '#233b5e'
      },
      {
        arrowColor: 'white',
        color: '#233b5e',
        maxRadius: (maxRadius - spacing * 3),
        points: 4,
        strokeColor: 'white'
      },
      {
        arrowColor: 'white',
        color: '#233b5e',
        maxRadius: (maxRadius - spacing * 2),
        points: 3,
        strokeColor: 'white'
      },
      {
        arrowColor: 'white',
        color: '#233b5e',
        maxRadius: (maxRadius - spacing * 1),
        points: 2,
        strokeColor: 'white'
      },
      {
        arrowColor: 'white',
        color: '#233b5e',
        maxRadius: maxRadius,
        points: 1,
        strokeColor: 'white'
      }
    ];
  }

  const OLYMPIC_TARGET = (spacing) => {
    return [
      {
        arrowColor: 'black',
        color: '#ffc107',
        maxRadius: (maxRadius - spacing * 10),
        points: 10,
        strokeColor: 'black',
      },
      {
        arrowColor: 'black',
        color: '#ffc107',
        maxRadius: (maxRadius - spacing * 9),
        points: 10,
        strokeColor: 'black'
      },
      {
        arrowColor: 'black',
        color: '#ffc107',
        maxRadius: (maxRadius - spacing * 8),
        points: 9,
        strokeColor: 'black'
      },
      {
        arrowColor: 'black',
        color: '#dc3545',
        maxRadius: (maxRadius - spacing * 7),
        points: 8,
        strokeColor: 'black'
      },
      {
        arrowColor: 'black',
        color: '#dc3545',
        maxRadius: (maxRadius - spacing * 6),
        points: 7,
        strokeColor: 'black'
      },
      {
        arrowColor: 'black',
        color: '#17a2b8',
        maxRadius: (maxRadius - spacing * 5),
        points: 6,
        strokeColor: 'black'
      },
      {
        arrowColor: 'black',
        color: '#17a2b8',
        maxRadius: (maxRadius - spacing * 4),
        points: 5,
        strokeColor: 'black'
      },
      {
        arrowColor: 'white',
        color: 'black',
        maxRadius: (maxRadius - spacing * 3),
        points: 4,
        strokeColor: 'white'
      },
      {
        arrowColor: 'white',
        color: 'black',
        maxRadius: (maxRadius - spacing * 2),
        points: 3,
        strokeColor: 'black'
      },
      {
        arrowColor: 'black',
        color: 'white',
        maxRadius: (maxRadius - spacing * 1),
        points: 2,
        strokeColor: 'black'
      },
      {
        arrowColor: 'black',
        color: 'white',
        maxRadius: maxRadius,
        points: 1,
        strokeColor: 'black'
      }
    ];
  }

  let canvasRef
  let context;
  let touchPos;
  let maxZoom = 2;
  let touchMove = false;
  let centerPoint = { x: canvasSizePx / 2, y: canvasSizePx / 2 };
  let touchCursorOffset = canvasSizePx / 3;

  const createArrow = (arrow) => {
    addArrow(arrow)
    renderTargetAndArrows()
  }

  const getTargetScoreByPosition = (position) => {
    let distanceFromCenter = Math.trunc(
      Math.sqrt(
        Math.pow(centerPoint.x - position.x, 2) +
        Math.pow(centerPoint.y - position.y, 2)
      )
    );
    for (let i = 0; i < targetScoreRange.length; i++) {
      const target = targetScoreRange[i];
      if (distanceFromCenter <= target.maxRadius) {
        return target;
      }
    }
    return null;
  }

  const drawArrow = (position, fill) => {
    let score;
    if (fill) {
      context.fillStyle = fill;
    } else {
      score = getTargetScoreByPosition(position);
      context.fillStyle = score ? score.arrowColor : '';
    }
    context.beginPath();
    context.arc(position.x, position.y, arrowRadius, 0, 2 * Math.PI);
    context.fill();
  }

  const handleMouseUp = (e) => {
    const pos = getMousePos(canvasRef, e);
    const newArrow = generateArrowOnTarget(pos, false);
    createArrow(newArrow);
  }

  const handleTouchEnd = (event) => {
    event.preventDefault();
    if (touchMove) {
      // don't trigger shots on ghost clicks, touch devices must pan a bit first
      const newArrow = generateArrowOnTarget(touchPos, true);
      createArrow(newArrow);
    }
    touchMove = false;
    touchPos = null;
    context.resetTransform();
    context.clearRect(0, 0, canvasSizePx, canvasSizePx);
    renderTargetAndArrows();
  }

  const generateArrowOnTarget = (zoomedCoordinates, isTouch) => {
    // all arrows are added in zoomed target, so we adjust position
    const normalCoordinates = isTouch
      ? {
        x: (zoomedCoordinates.x - zoomedCoordinates.x / maxZoom) * 2,
        y:
          (zoomedCoordinates.y -
            touchCursorOffset / maxZoom -
            zoomedCoordinates.y / maxZoom +
            touchCursorOffset / maxZoom) *
          2
      }
      : {
        x: (zoomedCoordinates.x - zoomedCoordinates.x / maxZoom) * 2,
        y: (zoomedCoordinates.y - zoomedCoordinates.y / maxZoom) * 2
      };

    const score = getTargetScoreByPosition(normalCoordinates);
    return {
      point: normalCoordinates,
      score: score ? score.points : 0
    };
  }

  const renderTargetAndArrows = () => {
    for (let i = targetScoreRange.length - 1; i >= 0; i--) {
      const score = targetScoreRange[i];
      context.fillStyle = score.color;
      context.strokeStyle = score.strokeColor;
      context.lineWidth = score.strokeWidth || strokeWidth;
      context.beginPath();
      context.arc(
        centerPoint.x,
        centerPoint.y,
        score.maxRadius,
        0,
        2 * Math.PI,
        false
      );
      context.fill();
      context.stroke();
    }
    // bulls eye
    context.fillStyle = 'black';
    context.beginPath();
    context.arc(centerPoint.x, centerPoint.y, 1, 0, 2 * Math.PI);
    context.fill();

    arrows.forEach(arrow => drawArrow(arrow.point));
  }

  // show zoomed canvas and pan it with cursor
  const handleMouseMove = (event) => {
    context.resetTransform();
    context.scale(maxZoom, maxZoom);
    context.clearRect(0, 0, canvasSizePx, canvasSizePx);
    const pos = getMousePos(canvasRef, event);
    context.translate(-pos.x / 2, -pos.y / 2);
    renderTargetAndArrows();
  }

  // show zoomed canvas on mobile
  const handleTouchMove = (e) => {
    // this is to prevent scrolling on iOS
    e.preventDefault();
    // Ignore multi touch events
    if (e.touches.length > 1) {
      return;
    }
    touchMove = true;
    context.resetTransform();
    context.scale(maxZoom, maxZoom);
    context.clearRect(0, 0, canvasSizePx, canvasSizePx);
    const pos = getTouchPos(canvasRef, e);
    touchPos = { x: pos.x, y: pos.y - touchCursorOffset };
    context.translate(-touchPos.x / 2, -touchPos.y / 2);
    renderTargetAndArrows();
    // draw crosshair for touch input
    drawArrow(touchPos, '#39ff14');
  }

  // get position of mouse cursor
  const getMousePos = (canvas, evt) => {
    const rect = canvas.getBoundingClientRect();
    return {
      x: evt.clientX - rect.left,
      y: evt.clientY - rect.top
    };
  }

  // get position for touch event
  const getTouchPos = (canvas, event) => {
    const rect = canvas.getBoundingClientRect();
    return {
      x: event.touches[0].clientX - rect.left,
      y: event.touches[0].clientY - rect.top
    };
  }

  const throttle = (cb, delay) => {
    let timesUp = true;
    return (event) => {
      if (!timesUp) return;
      setTimeout(function() {
        timesUp = true;
      }, delay);

      timesUp = false;
      cb(event);
    };
  }

  onMount(() => {
    getTarget()
    context = canvasRef.getContext('2d');
    // // touchmove is done via event listener to prevent window scrolling on ios
    canvasRef.addEventListener('touchmove', handleTouchMove);
    context.clearRect(0, 0, canvasSizePx, canvasSizePx);
    context.resetTransform();
    renderTargetAndArrows();
  })

  $: {
    if (canvasRef) {
      getTarget()
      context = canvasRef.getContext('2d');
      context.clearRect(0, 0, canvasSizePx, canvasSizePx);
      renderTargetAndArrows();
    }
  }
</script>

<div>
    <canvas
            bind:this={canvasRef}
            width={canvasSizePx}
            height={canvasSizePx}
            on:mouseenter={() => {
      context.scale(maxZoom, maxZoom);
      context.clearRect(0, 0, canvasSizePx, canvasSizePx);
      renderTargetAndArrows();
    }}
            on:mouseout={() => {
      context.resetTransform();
      context.clearRect(0, 0, canvasSizePx, canvasSizePx);
      renderTargetAndArrows();
    }}
            on:mouseup={event => {
      handleMouseUp(event)
    }}
            on:mousemove={throttle(event => handleMouseMove(event), 16)}
            on:touchend={event => handleTouchEnd(event)}
            on:touchstart={event => {
      context.resetTransform();
      context.scale(maxZoom, maxZoom);
      context.clearRect(0, 0, canvasSizePx, canvasSizePx);
      const pos = getTouchPos(canvasRef, event);
      const touchPos = { x: pos.x, y: pos.y - touchCursorOffset };
      context.translate(-touchPos.x / 2, -touchPos.y / 2);
      renderTargetAndArrows();
    }}
    >
    </canvas>

</div>

<style>

</style>