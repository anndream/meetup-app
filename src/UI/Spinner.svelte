<style>
  #loading {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 2rem;
  }

  .loading {
    display: inline-block;
    position: relative;
    width: 64px;
    height: 64px;
    transform: rotate(45deg);
    transform-origin: 32px 32px;
  }
  .loading div {
    top: 23px;
    left: 19px;
    position: absolute;
    width: 26px;
    height: 26px;
    background: #cf0056;
    animation: loading 1.2s infinite cubic-bezier(0.215, 0.61, 0.355, 1);
  }
  .loading div:after,
  .loading div:before {
    content: " ";
    position: absolute;
    display: block;
    width: 26px;
    height: 26px;
    background: #cf0056;
  }
  .loading div:before {
    left: -17px;
    border-radius: 50% 0 0 50%;
  }
  .loading div:after {
    top: -17px;
    border-radius: 50% 50% 0 0;
  }
  @keyframes loading {
    0% {
      transform: scale(0.95);
    }
    5% {
      transform: scale(1.1);
    }
    39% {
      transform: scale(0.85);
    }
    45% {
      transform: scale(1);
    }
    60% {
      transform: scale(0.95);
    }
    100% {
      transform: scale(0.9);
    }
  }
</style>

<div id="loading">
  <div class="loading">
    <div />
  </div>
</div>
