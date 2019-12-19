<template>
  <div id="app">
    <vs-navbar
      v-model="indexActive"
      color="rgb(31, 116, 255)"
      text-color="rgba(255,255,255,.1)"
      active-text-color="rgba(255,255,255,1)"
    >
      <div slot="title">
        <img class="logo" alt="logo" src="./assets/logo.png" />
      </div>
      <vs-navbar-item index="0">
        <a href="#">Subnet match</a>
      </vs-navbar-item>
    </vs-navbar>
    <div class="container">
      <vs-breadcrumb
        :items="[
          {
            title: 'Home',
            url: '/'
          },
          {
            title: 'Subnet match',

            active: true
          }
        ]"
      ></vs-breadcrumb>
      <vs-row vs-align="center" class="row">
        <vs-col
          vs-type="flex"
          vs-justify="center"
          vs-align="center"
          vs-lg="10"
          vs-sm="8"
          vs-xs="12"
        >
          <vs-input
            class="inputx"
            size="large"
            placeholder="input ip address for searching"
            style="width: 100%"
            v-model="searchIp"
          />
        </vs-col>
        <vs-col
          vs-type="flex"
          vs-justify="center"
          vs-align="center"
          vs-lg="2"
          vs-sm="4"
          vs-xs="12"
        >
          <vs-button
            color="primary"
            type="filled"
            @click="onSearch"
            style="width: 100%"
            size="large"
          >
            Search
          </vs-button>
        </vs-col>
      </vs-row>
      <vs-row vs-align="center" class="row">
        <vs-col
          vs-type="flex"
          vs-justify="center"
          vs-align="center"
          vs-lg="8"
          vs-sm="6"
          vs-xs="12"
        >
          <vs-input
            class="inputx"
            size="large"
            placeholder="input the url of the subnets file"
            style="width: 100%"
            v-model="subnetsUrl"
          />
        </vs-col>
        <vs-col
          vs-type="flex"
          vs-justify="center"
          vs-align="center"
          vs-lg="4"
          vs-sm="6"
          vs-xs="12"
        >
          <vs-button
            color="primary"
            type="filled"
            @click="onLoadSubnets"
            style="width: 100%"
            size="large"
          >
            Load subnets resources
          </vs-button>
        </vs-col>
      </vs-row>
      <vs-row vs-align="center" class="row">
        <vs-col vs-type="flex" vs-justify="center" vs-align="center" vs-w="12">
          <vs-textarea class="result-textarea" v-model="searchList" />
        </vs-col>
      </vs-row>
      <vs-row vs-align="center" class="row">
        <vs-col vs-type="flex" vs-justify="start" vs-align="center" vs-w="12">
          <vs-tooltip
            v-for="(item, index) in results"
            :key="index"
            title="详细信息"
            color="warning"
            :text="
              `起始地址: ${item.subnet.firstAddress}
              结束地址: ${item.subnet.lastAddress}
              广播地址: ${item.subnet.broadcastAddress}
              子网掩码: ${item.subnet.subnetMask}
              IP数量: ${item.subnet.numHosts}
              `
            "
          >
            <vs-chip color="success" transparent>
              {{ item.ip }}/{{ item.cidr || item.mask }} # {{ item.comments }}
            </vs-chip>
          </vs-tooltip>
        </vs-col>
      </vs-row>
    </div>
  </div>
</template>
<script>
/* eslint-disable no-console */
/* eslint-disable no-unused-vars */
/* eslint-disable no-useless-escape */
import { subnet, cidrSubnet } from "browserify-ip";
import logo from "./assets/logo.png";

export default {
  data: () => {
    return {
      searchIp: "",
      subnetsUrl: "",
      searchList: "",
      results: [
        // {
        //   ip: "202.203.208.33",
        //   cidr: "24",
        //   comments: "123",
        //   subnet: {
        //     networkAddress: "202.203.208.0",
        //     firstAddress: "202.203.208.1",
        //     lastAddress: "202.203.208.254",
        //     broadcastAddress: "202.203.208.255",
        //     subnetMask: "255.255.255.0",
        //     subnetMaskLength: 24,
        //     numHosts: 254,
        //     length: 256
        //   }
        // }
      ],
      indexActive: 0
    };
  },
  methods: {
    async onLoadSubnets() {
      try {
        const { subnetsUrl } = this;
        console.info(`onLoadSubnets`);
        const response = await fetch(subnetsUrl);
        // if HTTP-status is 200-299
        if (response.ok) {
          let searchList = await response.text();
          this.searchList = searchList;
        } else {
          this.$vs.notify({
            title: "Notification",
            text: `load resource failed with HTTP-Error: ${response.status}`,
            color: "warning"
          });
        }
      } catch (err) {
        console.info(err);
        this.$vs.notify({
          title: "Notification",
          text: `load resource failed with ${err}`,
          color: "danger"
        });
      }
    },
    onSearch() {
      const { searchIp, searchList } = this;
      // https://regex101.com/r/3ZI0Tq/1/
      const cidrSubnetRegexp = /^[^\S\r\n]*(?<ip>[\d.:a-f]{3,})(?:[\t\f \/,;]+)(?<cidr>\d{1,2})(?:[\t\f \/,;#]+)(?<comments>.*)$/gim;
      const maskSubnetRegexp = /^[^\S\r\n]*(?<ip>[\d.:a-f]{3,})(?:[\t\f \/,;]+)(?<mask>[\d.:a-f]{3,})(?:[\t\f \/,;#]+)(?<comments>.*)$/gim;
      const cidrSubnets = [];
      const maskSubnets = [];
      let matches = searchList.matchAll(cidrSubnetRegexp);
      for (const match of matches) {
        const {
          groups: { ip, cidr, comments }
        } = match;
        cidrSubnets.push({ ip, cidr, comments });
      }
      matches = searchList.matchAll(maskSubnetRegexp);
      for (const match of matches) {
        const {
          groups: { ip, mask, comments }
        } = match;
        maskSubnets.push({ ip, mask, comments });
      }
      console.info(
        `parsed subnets\ncidrSubnets: ${JSON.stringify(
          cidrSubnets,
          null,
          2
        )}\nmaskSubnets: ${JSON.stringify(maskSubnets, null, 2)}`
      );
      const matchedCidrSubnets = cidrSubnets.filter(item =>
        cidrSubnet(`${item.ip}/${item.cidr}`).contains(searchIp)
      );
      const matchedMaskSubnets = maskSubnets.filter(item =>
        subnet(item.ip, item.mask).contains(searchIp)
      );

      const results =
        [
          ...matchedCidrSubnets.map(item => ({
            ...item,
            subnet: cidrSubnet(`${item.ip}/${item.cidr}`)
          })),
          ...matchedMaskSubnets.map(item => ({
            ...item,
            subnet: subnet(item.ip, item.mask)
          }))
        ] || [];
      this.results = results;
    }
  }
};
</script>
<style>
#app .logo {
  width: 120px;
}

#app .container {
  margin: 0 50px;
}
#app .row {
  margin: 10px 0;
}

#app .result-textarea /deep/ .vs-textarea {
  font-size: 1em;
  height: 10rem;
  width: 100%;
}
#app .logo img {
  height: 100%;
  display: block;
}

#app .large-text {
  font-size: 1.25em;
}
</style>