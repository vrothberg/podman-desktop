<script lang="ts">
import { onMount } from 'svelte';
import StatusIcon from '../images/StatusIcon.svelte';
import DetailsPage from '../ui/DetailsPage.svelte';
import Tab from '../ui/Tab.svelte';
import StateChange from '../ui/StateChange.svelte';
import type { V1Ingress } from '@kubernetes/client-node';
import { stringify } from 'yaml';
import MonacoEditor from '../editor/MonacoEditor.svelte';
import type { IngressUI } from './IngressUI';
import { IngressRouteUtils } from './ingress-route-utils';
import { ingresses } from '/@/stores/ingresses';
import IngressRouteActions from './IngressRouteActions.svelte';
import ServiceDetailsSummary from './IngressRouteDetailsSummary.svelte';
import ServiceIcon from '../images/ServiceIcon.svelte';
import Route from '../../Route.svelte';

export let name: string;
export let namespace: string;

let ingressUI: IngressUI;
let detailsPage: DetailsPage;
let kubeService: V1Ingress | undefined;
let kubeError: string;

onMount(() => {
  const ingressRouteUtils = new IngressRouteUtils();

  return ingresses.subscribe(ingress => {
    const matchingIngress = ingress.find(srv => srv.metadata?.name === name && srv.metadata?.namespace === namespace);
    if (matchingIngress) {
      try {
        ingressUI = ingressRouteUtils.getIngressUI(matchingIngress);
        loadIngressDetails();
      } catch (err) {
        console.error(err);
      }
    } else if (detailsPage) {
      // the ingress has been deleted
      detailsPage.close();
    }
  });
});

async function loadIngressDetails() {
  const getKubeService = await window.kubernetesReadNamespacedIngress(ingressUI.name, namespace);
  if (getKubeService) {
    kubeService = getKubeService;
  } else {
    kubeError = `Unable to retrieve Kubernetes details for ${ingressUI.name}`;
  }
}
</script>

{#if ingressUI}
  <DetailsPage title="{ingressUI.name}" subtitle="{ingressUI.namespace}" bind:this="{detailsPage}">
    <StatusIcon slot="icon" icon="{ServiceIcon}" size="{24}" status="{ingressUI.status}" />
    <svelte:fragment slot="actions">
      <IngressRouteActions ingressRoute="{ingressUI}" detailed="{true}" on:update="{() => (ingressUI = ingressUI)}" />
    </svelte:fragment>
    <div slot="detail" class="flex py-2 w-full justify-end text-sm text-gray-700">
      <StateChange state="{ingressUI.status}" />
    </div>
    <svelte:fragment slot="tabs">
      <Tab title="Summary" url="summary" />
      <Tab title="Inspect" url="inspect" />
      <Tab title="Kube" url="kube" />
    </svelte:fragment>
    <svelte:fragment slot="content">
      <Route path="/summary" breadcrumb="Summary" navigationHint="tab">
        <ServiceDetailsSummary ingressRouteUI="{ingressUI}" ingressRoute="{kubeService}" kubeError="{kubeError}" />
      </Route>
      <Route path="/inspect" breadcrumb="Inspect" navigationHint="tab">
        <MonacoEditor content="{JSON.stringify(kubeService, undefined, 2)}" language="json" />
      </Route>
      <Route path="/kube" breadcrumb="Kube" navigationHint="tab">
        <MonacoEditor content="{stringify(kubeService)}" language="yaml" />
      </Route>
    </svelte:fragment>
  </DetailsPage>
{/if}
