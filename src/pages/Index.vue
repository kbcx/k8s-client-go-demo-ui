<!-- eslint-disable @typescript-eslint/no-explicit-any -->
<template>
  <q-page class="q-pa-md">
    <div class="row justify-center">
      <div class="col-12">
        <div class="q-gutter-md" style="max-width: 300px">
          <q-select
            v-model="namespaceName"
            :options="namespacesOptions"
            label="Namespaces"
            @update:model-value="ListPods()"
          />

          <q-select
            v-model="podName"
            :options="podOptions"
            label="Pods"
            @update:model-value="ListContainers()"
          />

          <q-select
            v-model="containerName"
            :options="containerOptions"
            label="Containers"
            @update:model-value="WebsocketTerminal()"
          />
        </div>
      </div>
      <div class="col-12">
        <XtermComponent
          :active="xtermActive"
          :namespaceName="namespaceName"
          :podName="podName"
          :containerName="containerName"
          :command="command"
        />
      </div>
    </div>
  </q-page>
</template>

<script lang="ts">
import XtermComponent from 'components/XtermComponent.vue';
import { useQuasar } from 'quasar';
import { defineComponent, onMounted, ref } from 'vue';
import { api } from 'boot/axios';

export default defineComponent({
  name: 'PageIndex',
  components: { XtermComponent },

  setup() {
    const $q = useQuasar();
    const xtermActive = ref(false);
    const namespacesOptions = ref([] as string[]);
    const podOptions = ref([] as string[]);
    const containerOptions = ref([] as string[]);

    let namespaceName = ref('');
    let podName = ref('');
    let containerName = ref('');
    let command = ref('sh');

    async function ListNamespaces() {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      var namespaceList = [] as any[];
      await api
        .get('/namespaces')
        .then((response) => {
          console.log('call /namespaces resp is', response.data);
          // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment
          namespaceList = response.data;
        })
        .catch(() => {
          $q.notify({
            color: 'negative',
            position: 'top',
            message: 'Loading /namespaces failed',
            icon: 'report_problem',
          });
        });
      // eslint-disable-next-line @typescript-eslint/no-unsafe-return
      return namespaceList;
    }

    async function ListPods() {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      var podList = [] as any[];
      await api
        .get('/namespace/' + namespaceName.value + '/pods')
        .then((response) => {
          console.log(
            'call /namespace/',
            namespaceName.value,
            '/pods resp is',
            response.data
          );
          // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment
          podList = response.data;
        })
        .catch(() => {
          $q.notify({
            color: 'negative',
            position: 'top',
            message: 'call /namespace/' + namespaceName.value + '/pods failed',
            icon: 'report_problem',
          });
        });
      // eslint-disable-next-line @typescript-eslint/no-unsafe-return
      let pods: string[] = [];
      for (var i = 0; i < podList.length; i++) {
        // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access, @typescript-eslint/no-unsafe-argument
        pods.push(podList[i]['metadata']['name']);
      }
      podOptions.value = pods;
    }

    async function ListContainers() {
      // eslint-disable-next-line @typescript-eslint/no-explicit-any
      var podDetail = {} as { spec: { containers: any[] } };
      await api
        .get('/namespace/' + namespaceName.value + '/pod/' + podName.value)
        .then((response) => {
          console.log(
            'call /namespace/',
            namespaceName.value,
            '/pod/',
            podName.value,
            ' resp is',
            response.data
          );
          // eslint-disable-next-line @typescript-eslint/no-unsafe-assignment
          podDetail = response.data;
        })
        .catch(() => {
          $q.notify({
            color: 'negative',
            position: 'top',
            message:
              'call /namespace/' +
              namespaceName.value +
              '/pod/' +
              podName.value +
              ' failed',
            icon: 'report_problem',
          });
        });
      // eslint-disable-next-line @typescript-eslint/no-unsafe-return
      let containers: string[] = [];
      for (var i = 0; i < podDetail['spec']['containers'].length; i++) {
        // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access, @typescript-eslint/no-unsafe-argument
        containers.push(podDetail['spec']['containers'][i]['name']);
      }
      containerOptions.value = containers;
    }

    function WebsocketTerminal() {
      xtermActive.value = true;
      return;
    }

    onMounted(async () => {
      var result = await ListNamespaces();
      console.log('namespaces:', result);
      let namespaces: string[] = [];
      for (var i = 0; i < result.length; i++) {
        // eslint-disable-next-line @typescript-eslint/no-unsafe-member-access, @typescript-eslint/no-unsafe-argument
        namespaces.push(result[i]['metadata']['name']);
      }
      console.log('namespaces Options:', namespaces);
      namespacesOptions.value = namespaces;
    });

    return {
      xtermActive,
      namespaceName,
      namespacesOptions,
      ListPods,
      podName,
      podOptions,
      ListContainers,
      containerName,
      containerOptions,
      command,
      WebsocketTerminal,
    };
  },
});
</script>
