<script setup lang="ts">
import type { FormSubmitEvent } from "@nuxt/ui";
import { useMutation } from "@tanstack/vue-query";

const meta = useProjectCreateMeta();
const modelValue = defineModel<ProjectCreateEval>({
  default: () => {
    return {};
  },
});

const props = withDefaults(
  defineProps<{
    showBackButton?: boolean;
    nextButtonLabel?: string;
    inferenceModel?: string[];
    embeddingModel?: string[];
    region?: string;
  }>(),
  {
    showBackButton: true,
    nextButtonLabel: "Next",
    inferenceModel: [],
    embeddingModel: [],
  }
);

const state = reactive<Partial<ProjectCreateEval>>({
  service: modelValue.value?.service || undefined,
  ragas_embedding_llm: modelValue.value?.ragas_embedding_llm || undefined,
  ragas_inference_llm: modelValue.value?.ragas_inference_llm || undefined,
  guardrails: modelValue.value?.guardrails || [],
});

const emits = defineEmits(["next", "previous"]);

const onSubmit = (event: FormSubmitEvent<ProjectCreateEval>) => {
  event.data['guardrails'] = event.data['guardrails'].filter(guardrail=>guardrail.name!=='None')
  modelValue.value = event.data;
  emits("next");
};

const guardrailsList = ref([]);

const { mutateAsync: getGuardrailsList, isPending: isFetchingGuardrailsList } =
  useMutation({
    mutationFn: async () => {
      const response = await useGuardrailsList();
      guardrailsList.value = response?.map((item) => {
        return {
          label: item.name + ' - ' + item.version,
          value: item.name,
          name: item.name,
          guardrails_id: item.guardrails_id,
          guardrail_version: item.version,
          guardrail_description : item.description,
          enable_prompt_guardrails: true,
          enable_context_guardrails: true,
          enable_response_guardrails: true,
        };
      });
      guardrailsList.value.unshift({
          label: 'None',
          value: 'none',
          name: 'None',
          guardrails_id: null,
          guardrail_version: null,
          guardrail_description : null,
          enable_prompt_guardrails: true,
          enable_context_guardrails: true,
          enable_response_guardrails: true,
      })
      return response;
    },
  });

const fetchGuardrails = () => {
  state.guardrails = [];
  getGuardrailsList();
};

onMounted(() => {
  fetchGuardrails();
});
</script>

<template>
  <UForm
    :schema="ProjectCreateEvalSchema"
    :state="state"
    :validate-on="['input']"
    @submit="onSubmit"
  >
    <UCard class="w-full">
      <label class="font-bold text-sm -my-4">Guardrails {{ state?.guardrails?.length ? `(${state?.guardrails?.length})` : ''}} - Applied at User Query,Context and Model Response levels</label>
      <div class="flex gap-2">
        <UFormField
          name="guardrails"
          class="text-ellipsis overflow-hidden flex-11"
        >
          <USelectMenu
            v-model="state.guardrails"
            :disabled="isFetchingGuardrailsList"
            :loading="isFetchingGuardrailsList"
            multiple
            :items="guardrailsList"
            class="w-full my-7"
            placeholder="None"
          >
          
            <template #item-label="{ item }">
                {{ item.label + `${item.guardrail_description ?  ' - ( ' + item.guardrail_description + ')' : ''}` }}
            </template>
          </USelectMenu>
         <div class="my-0">
          <ULink class="text-blue-500 hover:underline" target="_blank" raw :to="`https://${props.region}.console.aws.amazon.com/bedrock/home?region=${props.region}#/guardrails`" active-class="font-bold" inactive-class="text-[var(--ui-text-muted)]">Create guardrails on AWS</ULink>
        </div>
        </UFormField>
        
        <UFormField name="refetch_guardrail_list" label=" " class="flex-1">
          <UButton
            label="Fetch Guardrails"
            trailing-icon="i-lucide-repeat-2"
            @click.prevent="fetchGuardrails"
          />
          <template #hint>
            <FieldTooltip field-name="guardrails" />
          </template>
        </UFormField>
      </div>
    </UCard>
    <UCard>
    <label class="font-bold text-sm">Evaluation</label>
    <div class="my-3">
    <UFormField name="service" :label="`Service`" required>
      <USelectMenu
        v-model="state.service"
        value-key="value"
        :items="meta.evalStrategy.service"
        class="w-full"
      />
      <template #hint>
        <FieldTooltip field-name="service" />
      </template>
    </UFormField>
    <UFormField
      name="ragas_embedding_llm"
      :label="`Embedding Model`"
      required
    >
      <USelectMenu
        v-model="state.ragas_embedding_llm"
        value-key="value"
        :items="useFilteredRagasEmbeddingModels(embeddingModel)"
        class="w-full"
      />
      <template #hint>
        <FieldTooltip field-name="ragas_embedding_llm" />
      </template>
    </UFormField>
    <UFormField
      name="ragas_inference_llm"
      :label="`Inferencing Model`"
      required
    >
      <USelectMenu
        v-model="state.ragas_inference_llm"
        value-key="value"
        :items="useFilteredRagasInferenceModels(inferenceModel)"
        class="w-full"
      />
      <template #hint>
        <FieldTooltip field-name="ragas_inference_llm" />
      </template>
    </UFormField>
  </div>
    </UCard>
    <div class="flex justify-between items-center w-full mt-6">
      <div>
        <UButton
          v-if="showBackButton"
          type="button"
          icon="i-lucide-arrow-left"
          label="Back"
          variant="outline"
          @click.prevent="emits('previous')"
        />
      </div>
      <div>
        <UButton
          trailing-icon="i-lucide-arrow-right"
          :label="nextButtonLabel"
          type="submit"
        />
      </div>
    </div>
  </UForm>
</template>
