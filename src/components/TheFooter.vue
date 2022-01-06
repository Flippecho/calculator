<template>
  <div id="footer" class="fixed-bottom">
    <p id="quote">:D 正在加载...</p>
    <p id="copyright">© 8208201030 &nbsp; 张岩峰</p>
  </div>
</template>

<script>
fetch('https://v1.hitokoto.cn/?c=d&c=i&c=k')
    .then(response => response.json())
    .then(data => {
      const hitokoto = document.getElementById('quote');
      hitokoto.innerText = data.hitokoto + ' —— ' + data.from;
    })
    .catch(console.error)

export default {
  name: "TheFooter"
}
</script>

<style scoped>
#footer {
  width: 90vw;
  left: 5vw;
  font-size: 1.5em;
}

#quote {
  display: table-cell;
  float: left;
  width: 60vw;
  text-align: center;
  white-space: nowrap;
  overflow: auto;
}

#copyright {
  display: table-cell;
  float: right;
  width: 20vw;
  text-align: center;
  white-space: nowrap;
  overflow: auto;
}
</style>