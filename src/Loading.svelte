<div class="loading">
    <div style="animation-delay: 0.2s;"></div>
    <div style="animation-delay: 0.4s;"></div>
    <div style="animation-delay: 0.6s;"></div>
</div>
<style>
.loading {
    width: 20px;
    display: flex;
    gap: 2px;
    justify-content: center;
    align-items: center;
}
.loading > div {
    width: 3px;
    height: 3px;
    animation: loading 1.5s infinite ease-in-out;
    transform: scale(1);
    transform-origin: center;
    clip-path: circle();
    background: black;
    opacity: 0;
}
@keyframes loading {
    0% {
        transform: scale(1);
        opacity: 0;
    } 
    30% {
        transform: scale(1.4);
        opacity: 1;
    } 
    40% {
        transform: scale(1.1);
    }
    60% {
        transform: scale(1);
        opacity: 0;
    }
}
</style>