<script setup>
import { useNav } from "@slidev/client";

const nav = useNav();

addEventListener("gamepadconnected", (evt) => {
  const gp = evt.gamepad;
  const pressed = {
    left: gp.buttons[4].pressed,
    right: gp.buttons[5].pressed,
  };
  const loop = () => {
    const newPressed = {
      left: gp.buttons[4].pressed,
      right: gp.buttons[5].pressed,
    };
    if (newPressed.left && !pressed.left) {
      nav.prev();
    }
    if (newPressed.right && !pressed.right) {
      nav.next();
    }
    Object.assign(pressed, newPressed);
    requestAnimationFrame(loop);
  };
  loop();
});
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=VT323&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Share+Tech&display=swap");

* {
  font-family: "Share Tech", serif;
  font-weight: 400;
  font-style: normal;
}

strong {
  color: yellow;
}
</style>
