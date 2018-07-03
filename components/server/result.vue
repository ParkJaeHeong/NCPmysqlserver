<template>
  <section>

    <!-- <header class="text-center pv-md">
      <h2>서버 생성 결과</h2>
      <p class="mt-10 color-gray-light">생성된 VM 서버 결과입니다</p>
    </header> -->

    <h4>서버 생성 결과</h4>
    <p class="mt-10 color-gray">생성된 VM 서버 결과입니다</p>
    <!-- <hr class="mv-30" style="border-color: #000;"> -->

    <table class="table-server mt-30" v-if="servers.length">
      <tbody>
        <tr v-for="(server, index) in servers" :key="index">
          <td class="text-center">
            <img src="/img/server.gif" alt="server icon" style="width:50px">
            <p class="mt-5">{{server.name}}</p>
          </td>
          <td>
            <b>Ping</b>
            <p v-for="(master, index2) in server.masters" :key="index2" class="mt-5">
              Server_id: {{master.Server_id}}<br>
              Host: {{master.Host}}<br>
              Slave_UUID: {{master.Slave_UUID}}<br>
              Port: {{master.Port}}<br>
              Master_id: {{master.Master_id}}<br>
            </p>
            <p v-for="(slave, index2) in server.slaves" :key="index2" class="mt-5">
              Slave_IO_Running: {{slave.Slave_IO_Running}}<br>
              Seconds_Behind_Master: {{slave.Seconds_Behind_Master}}<br>
              Slave_SQL_Running: {{slave.Slave_SQL_Running}}<br>
              Slave_id: {{slave.Slave_id}}<br>
            </p>
          </td>
          <td>
            <div class="server-status" :class="{ 'off': !server.status }"></div>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="mt-30 text-center">
      <nuxt-link :to="{ name: 'server' }" class="btn btn-primary">서버 세팅 추가하기</nuxt-link>
    </div>

  </section>
</template>

<script>
export default {
  props: {
    masterInfo: {
      type: Array,
      default: function () {
        return [
          {'Server_id': '3', 'Host': 'nbp003', 'Slave_UUID': '2db51418-7adb-11e8-ae5d-f220cde1d013', 'Port': '3306', 'Master_id': '1'},
          {'Server_id': '2', 'Host': 'nbp002', 'Slave_UUID': '2db93880-7adb-11e8-b9a1-f220cd40d0bb', 'Port': '3306', 'Master_id': '1'},
          {'Server_id': '4', 'Host': 'nbp004', 'Slave_UUID': '2dd9904c-7adb-11e8-ae4b-f220cd19108a', 'Port': '3306', 'Master_id': '3'}
        ]
      }
    },
    slaveInfo: {
      type: Array,
      default: function () {
        return [
          {'Slave_IO_Running': 'Yes', 'Seconds_Behind_Master': '0', 'Slave_SQL_Running': 'Yes', 'Slave_id': '2'},
          {'Slave_IO_Running': 'Yes', 'Seconds_Behind_Master': '0', 'Slave_SQL_Running': 'Yes', 'Slave_id': '3'},
          {'Slave_IO_Running': 'Yes', 'Seconds_Behind_Master': '0', 'Slave_SQL_Running': 'Yes', 'Slave_id': '4'}
        ]
      }
    }
  },
  data () {
    return {
      servers: []
      // masterInfo: null,
      // slaveInfo: null
      // masterInfo: [
      //   {'Server_id': '3', 'Host': 'nbp003', 'Slave_UUID': '2db51418-7adb-11e8-ae5d-f220cde1d013', 'Port': '3306', 'Master_id': '1'},
      //   {'Server_id': '2', 'Host': 'nbp002', 'Slave_UUID': '2db93880-7adb-11e8-b9a1-f220cd40d0bb', 'Port': '3306', 'Master_id': '1'},
      //   {'Server_id': '4', 'Host': 'nbp004', 'Slave_UUID': '2dd9904c-7adb-11e8-ae4b-f220cd19108a', 'Port': '3306', 'Master_id': '3'}
      // ],
      // slaveInfo: [
      //   {'Slave_IO_Running': 'Yes', 'Seconds_Behind_Master': '0', 'Slave_SQL_Running': 'Yes', 'Slave_id': '2'},
      //   {'Slave_IO_Running': 'Yes', 'Seconds_Behind_Master': '0', 'Slave_SQL_Running': 'Yes', 'Slave_id': '3'},
      //   {'Slave_IO_Running': 'Yes', 'Seconds_Behind_Master': '0', 'Slave_SQL_Running': 'Yes', 'Slave_id': '4'}
      // ]
    }
  },
  created: function () {
    // let result = this.$store.state.result
    // if (result.masterInfo) {
    //   this.masterInfo = result.masterInfo
    //   this.slaveInfo = result.slaveInfo
    //   this.setServerArr()
    //   this.$store.commit('set_result', null)
    // } else {
    //   this.$router.push('/server')
    // }
    this.setServerArr()
  },
  methods: {
    setServerArr () {
      let tempArr = []
      let tempName = ''
      this.masterInfo.forEach((mInfo, index) => {
        if (tempArr.indexOf(Number(mInfo.Master_id)) === -1) {
          tempArr.push(Number(mInfo.Master_id))
          if (index === 0) tempName = mInfo.Host.substring(0, mInfo.Host.length - 1)
        }
      })
      this.slaveInfo.forEach(sInfo => {
        if (tempArr.indexOf(Number(sInfo.Slave_id)) === -1) {
          tempArr.push(Number(sInfo.Slave_id))
        }
      })
      tempArr.sort()
      tempArr.forEach(index => {
        this.servers.push({
          id: `${index}`,
          name: `${tempName}${index}`,
          masters: [],
          slaves: [],
          status: true
        })
      })
      this.setMaserInfo()
      this.setSlaveInfo()
    },
    setMaserInfo () {
      this.masterInfo.forEach(mInfo => {
        if (mInfo) this.servers[mInfo.Master_id - 1].status = true
        this.servers[mInfo.Master_id - 1].masters.push(mInfo)
      })
    },
    setSlaveInfo () {
      this.slaveInfo.forEach(sInfo => {
        this.servers[sInfo.Slave_id - 1].slaves.push(sInfo)
        if (sInfo.Slave_IO_Running === 'Yes' && sInfo.Seconds_Behind_Master === '0' && sInfo.Slave_SQL_Running === 'Yes') {
          this.servers[sInfo.Slave_id - 1].status = true
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
  @import "~assets/less/bootstrap/variables.less";

  .table-server {
    width: 100%;
    border-top: 1px solid @brand-primary;
    th, td {
      padding: 20px;
      border-bottom: 1px solid #ddd;
    }
  }

  .server-status {
      width: 25px;
      height: 25px;
      border-radius: 50%;
      background-color: @brand-success;
      &.off {
        background-color: @brand-danger;
      }
  }
</style>
