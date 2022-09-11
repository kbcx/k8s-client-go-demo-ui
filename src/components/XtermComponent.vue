<template>
  <div>
    <div>xterm of {{ namespaceName }}/{{ podName }}/{{ containerName }}</div>
    <div class="">
      <q-btn color="primary" label="Get XTerm" @click="getXterm" />
    </div>
    <div id="terminal"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onBeforeUnmount, onMounted } from 'vue';

import { Terminal } from 'xterm';
import { FitAddon } from 'xterm-addon-fit';

export default defineComponent({
  name: 'XtermComponent',
  props: {
    active: {
      type: Boolean,
      required: true,
      default: false,
    },
    namespaceName: {
      type: String,
      required: true,
    },
    podName: {
      type: String,
      required: true,
    },
    containerName: {
      type: String,
      required: true,
    },
    command: {
      type: String,
      required: true,
      default: () => 'sh',
    },
  },
  setup(props) {
    let socket: WebSocket;
    let term: Terminal;
    let fitAddon: FitAddon;

    function GetSocketUrl() {
      return (
        'ws://127.0.0.1:8080/namespace/' +
        props.namespaceName +
        '/pod/' +
        props.podName +
        '/container/' +
        props.containerName +
        '?command=' +
        props.command
      );
    }

    // function getColsAndRows(ele: HTMLElement) {
    //   // 暂时不用
    //   ele = ele || document.getElementById('terminal');
    //   if (ele == null) {
    //     return {
    //       rows: 50,
    //       cols: 25,
    //     };
    //   } else {
    //     return {
    //       rows: ele.clientHeight / 15, // 50
    //       cols: ele.clientWidth / 5, // 25
    //     };
    //   }
    // }

    function resizeTerm() {
      fitAddon.fit();
      term.scrollToBottom();
    }

    function initTerminal() {
      let ele: HTMLElement | null = document.getElementById('terminal');
      if (ele == null) {
        console.log('get terminal err');
        return;
      }
      term = new Terminal({
        cursorBlink: true,
        cursorStyle: 'underline', // 'block' | 'underline' | 'bar'
        scrollback: 100,
      });
      // 设置了cols或者fitAddon.fit(); 当一行字符超过80个时会替换现在的内容，否则换行
      fitAddon = new FitAddon();
      term.loadAddon(fitAddon);
      term.open(ele);
      // 自适应大小(使终端的尺寸和几何尺寸适合于终端容器的尺寸)，初始化的时候宽高都是对的
      fitAddon.fit();
      term.focus();
      term.onData((data) => {
        var msg = { type: 'input', input: data };
        socket.send(JSON.stringify(msg));
      });

      window.addEventListener('resize', resizeTerm);
    }

    function socketOnOpen() {
      socket.onopen = () => {
        initTerminal();
      };
    }
    function socketOnMessage() {
      socket.onmessage = (event: MessageEvent) => {
        // 接收推送的消息
        // eslint-disable-next-line @typescript-eslint/no-unsafe-argument
        term.write(event.data);
      };
    }
    function socketOnClose() {
      socket.onclose = () => {
        console.log('close socket');
      };
    }
    function socketOnError() {
      socket.onerror = (err) => {
        console.log('socket error', err);
      };
    }

    onMounted(() => {
      console.log('xterm is mounted.');
    });

    function getXterm() {
      if (props.active == false) {
        console.log('some params is missing.');
        return;
      }
      socket = new WebSocket(GetSocketUrl());
      socketOnClose();
      socketOnOpen();
      socketOnError();
      socketOnMessage();
    }

    onBeforeUnmount(() => {
      socket.close();
      term && term.dispose();
    });

    return {
      getXterm,
    };
  },
});
</script>

<style scoped>
#terminal {
  padding: 15px 0;
}
</style>
