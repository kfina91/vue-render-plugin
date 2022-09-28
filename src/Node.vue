<template>
  <div class="node" :class="className" :style="{background: node.ref.color, border: node.ref.borderColor}" @mouseover="hovered=true" @mouseout="hovered=false">
    <div class="title">{{node.ref.displayName || node.name}}</div>
    <!-- Outputs-->
    <div class="output" v-for="output in outputs()" :key="output.key">
      <div class="output-title">{{ output.name }}</div>
      <Socket
        v-socket:output="output"
        type="output"
        :socket="output.socket"
      ></Socket>
    </div>
    <!-- Controls-->
    <div
      class="control"
      v-for="control in controls()"
      :key="control.key"
      v-control="control"
    >
    </div>
    <!-- Inputs-->
    <div v-if="inputs().length > 0" class="innertext">Input parameters:</div>
    <div class="input" v-for="input in inputs()" :key="input.key">
      <Socket
        v-socket:input="input"
        type="input"
        :socket="input.socket"
      ></Socket>
      <div class="input-title" v-show="!input.showControl()">
        {{ input.name }}
      </div>
      <div
        class="input-control"
        v-show="input.showControl()"
        v-control="input.control"
      ></div>
    </div>
  </div>
</template>

<script>
import { defineComponent, computed, ref } from "vue";
import Mixin from './mixin';
import Socket from "./Socket.vue";
import { kebab } from "./utils";

// source: https://stackoverflow.com/questions/5560248/programmatically-lighten-or-darken-a-hex-color-or-rgb-and-blend-colors
// Version 4.0
/* const pSBCr=(d)=>{
  let n=d.length,x={};
  if(n>9){
    [r,g,b,a]=d=d.split(","),n=d.length;
    if(n<3||n>4)return null;
    x.r=i(r[3]=="a"?r.slice(5):r.slice(4)),x.g=i(g),x.b=i(b),x.a=a?parseFloat(a):-1
  }else{
    if(n==8||n==6||n<4)return null;
    if(n<6)d="#"+d[1]+d[1]+d[2]+d[2]+d[3]+d[3]+(n>4?d[4]+d[4]:"");
    d=i(d.slice(1),16);
    if(n==9||n==5)x.r=d>>24&255,x.g=d>>16&255,x.b=d>>8&255,x.a=m((d&255)/0.255)/1000;
    else x.r=d>>16,x.g=d>>8&255,x.b=d&255,x.a=-1
  }return x};
const pSBC=(p,c0,c1,l)=>{
    let r,g,b,P,f,t,h,i=parseInt,m=Math.round,a=typeof(c1)=="string";
    if(typeof(p)!="number"||p<-1||p>1||typeof(c0)!="string"||(c0[0]!='r'&&c0[0]!='#')||(c1&&!a))return null;
    h=c0.length>9,h=a?c1.length>9?true:c1=="c"?!h:false:h,f=pSBCr(c0),P=p<0,t=c1&&c1!="c"?pSBCr(c1):P?{r:0,g:0,b:0,a:-1}:{r:255,g:255,b:255,a:-1},p=P?p*-1:p,P=1-p;
    if(!f||!t)return null;
    if(l)r=m(P*f.r+p*t.r),g=m(P*f.g+p*t.g),b=m(P*f.b+p*t.b);
    else r=m((P*f.r**2+p*t.r**2)**0.5),g=m((P*f.g**2+p*t.g**2)**0.5),b=m((P*f.b**2+p*t.b**2)**0.5);
    a=f.a,t=t.a,f=a>=0||t>=0,a=f?a<0?t:t<0?a:a*P+t*p:0;
    if(h)return"rgb"+(f?"a(":"(")+r+","+g+","+b+(f?","+m(a*1000)/1000:"")+")";
    else return"#"+(4294967296+r*16777216+g*65536+b*256+(f?m(a*255):0)).toString(16).slice(1,f?undefined:-2)
} */

export default defineComponent({
  name: "node",
  components: {
    Socket
  },
  mixins: [Mixin],
  setup(props) {
    const hovered = ref(false);
    const selected = () => {
      return props.editor.selected.contains(props.node) ? "selected" : "";
    };
    const className = computed(() => {
      return kebab([selected(), props.node.name]);
    });
    // for brightened hover effect via computed
    /*const bgcolor = computed(() => {
      if (hovered.value) pSBC(0.25, props.node.ref.color); // 25% brighter version of the color
      else return props.node.ref.color;
    });*/
    // console.log("props.node", props.node);

    return { className, hovered, "node": props.node };
  }
});
</script>

<style scoped lang="scss">
@use 'sass:math';
@import "./vars";

.node {
  background: $node-color;
  border-width: 2px;
  border-style: solid;
  border-radius: 10px;
  cursor: pointer;
  min-width: $node-width;
  height: auto;
  padding-bottom: 6px;
  box-sizing: content-box;
  position: relative;
  user-select: none;
  .title {
    color: white;
    font-size: 18px;
    padding: 8px;
  }
  .innertext {
    color: white;
    font-size: 14px;
    padding: 4px;
    padding-left: 10px;
  }
  .output {
    text-align: right;
  }
  .input {
    text-align: left;
  }
  .input-title,.output-title {
    color: white;
    vertical-align: middle;
    display: inline-block;
    font-size: 14px;
    line-height: $socket-size;
  }
  .input-control {
    z-index: 1;
    width: calc(100% - #{$socket-size + 2*$socket-margin});
    vertical-align: middle;
    display: inline-block;
  }
  .control {
    padding: $socket-margin math.div($socket-size,2) + $socket-margin;
  }
}
</style>
