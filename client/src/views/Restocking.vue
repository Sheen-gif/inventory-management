<template>
  <div class="restocking">
    <div class="page-header">
      <h2>Restocking Planner</h2>
      <p>Set your available budget, generate restock recommendations from demand forecasts, and place an order.</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <!-- Budget Card -->
      <div class="card budget-card">
        <div class="card-header">
          <h3 class="card-title">Available Budget</h3>
        </div>
        <div class="budget-control">
          <div class="budget-display">${{ budget.toLocaleString() }}</div>
          <input
            type="range"
            min="10000"
            max="500000"
            step="5000"
            v-model.number="budget"
            class="budget-slider"
          />
          <div class="budget-range-labels">
            <span>$10,000</span>
            <span>$500,000</span>
          </div>
        </div>
        <div class="budget-actions">
          <button class="btn-calculate" @click="calculateRecommendations">
            Calculate Recommendations
          </button>
          <span class="priority-note">Increasing trend items are prioritized first. Decreasing trend items are excluded.</span>
        </div>
      </div>

      <!-- Stats Row -->
      <div v-if="hasCalculated" class="stats-grid">
        <div class="stat-card">
          <div class="stat-label">Items Recommended</div>
          <div class="stat-value">{{ recommendations.length }}</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">Total Estimated Cost</div>
          <div class="stat-value">${{ totalCost.toLocaleString(undefined, { maximumFractionDigits: 0 }) }}</div>
        </div>
        <div class="stat-card" :class="{ 'stat-card-warning': remainingBudget < 0 }">
          <div class="stat-label">Remaining Budget</div>
          <div class="stat-value" :class="{ 'negative': remainingBudget < 0 }">
            ${{ remainingBudget.toLocaleString(undefined, { maximumFractionDigits: 0 }) }}
          </div>
        </div>
      </div>

      <!-- Recommendations Table -->
      <div v-if="hasCalculated" class="card">
        <div class="card-header">
          <h3 class="card-title">Restock Recommendations ({{ recommendations.length }})</h3>
        </div>

        <div v-if="recommendations.length === 0" class="empty-state">
          No items fit within the current budget. Try increasing the budget.
        </div>

        <div v-else class="table-container">
          <table class="restocking-table">
            <thead>
              <tr>
                <th>SKU</th>
                <th>Item Name</th>
                <th>Trend</th>
                <th>Forecasted Demand</th>
                <th>Qty to Order</th>
                <th>Unit Cost</th>
                <th>Estimated Cost</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="rec in recommendations" :key="rec.item_sku">
                <td><strong>{{ rec.item_sku }}</strong></td>
                <td>
                  {{ rec.item_name }}
                  <span v-if="rec.partial" class="partial-badge">Partial</span>
                </td>
                <td>
                  <span :class="['badge', rec.trend]">{{ rec.trend }}</span>
                </td>
                <td>{{ rec.forecasted_demand }}</td>
                <td>
                  <input
                    type="number"
                    class="qty-input"
                    min="1"
                    :value="adjustments[rec.item_sku] ?? rec.qty"
                    @input="adjustments[rec.item_sku] = Math.max(1, parseInt($event.target.value) || 1)"
                  />
                </td>
                <td>${{ rec.unitCost.toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 }) }}</td>
                <td><strong>${{ ((adjustments[rec.item_sku] ?? rec.qty) * rec.unitCost).toLocaleString(undefined, { maximumFractionDigits: 0 }) }}</strong></td>
              </tr>
            </tbody>
            <tfoot>
              <tr>
                <td colspan="6" class="total-label">Total</td>
                <td class="total-value">${{ totalCost.toLocaleString(undefined, { maximumFractionDigits: 0 }) }}</td>
              </tr>
            </tfoot>
          </table>
        </div>

        <div class="order-actions">
          <button
            class="btn-primary"
            :disabled="recommendations.length === 0 || isSubmitting || totalCost > budget"
            @click="placeOrder"
          >
            {{ isSubmitting ? 'Placing Order...' : 'Place Restocking Order' }}
          </button>
          <span v-if="totalCost > budget && recommendations.length > 0" class="order-message warning">
            Adjust quantities — total exceeds budget.
          </span>
          <span v-if="orderMessage" :class="['order-message', orderSuccess ? 'success' : 'error']">
            {{ orderMessage }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'

// Cost estimates based on SKU prefix for forecast items that don't match inventory SKUs
const DEFAULT_COSTS = {
  WDG: 45.00,
  BRG: 85.00,
  GSK: 12.50,
  MTR: 350.00,
  FLT: 8.75,
  VLV: 125.00,
  PSU: 65.00,
  SNR: 89.50,
  CTL: 22.00
}
const FALLBACK_COST = 50.00

export default {
  name: 'Restocking',
  setup() {
    const { t } = useI18n()
    const loading = ref(true)
    const error = ref(null)
    const budget = ref(100000)
    const allForecasts = ref([])
    const inventoryMap = ref({})
    const recommendations = ref([])
    const adjustments = ref({})
    const hasCalculated = ref(false)
    const isSubmitting = ref(false)
    const orderMessage = ref('')
    const orderSuccess = ref(false)

    const getUnitCost = (sku) => {
      if (inventoryMap.value[sku] !== undefined) return inventoryMap.value[sku]
      const prefix = sku.split('-')[0]
      return DEFAULT_COSTS[prefix] ?? FALLBACK_COST
    }

    const calculateRecommendations = () => {
      // Priority: increasing first, then stable; decreasing items excluded
      const prioritized = [
        ...allForecasts.value.filter(f => f.trend === 'increasing'),
        ...allForecasts.value.filter(f => f.trend === 'stable')
      ]

      let remaining = budget.value
      const result = []

      for (const forecast of prioritized) {
        if (remaining <= 0) break
        const unitCost = getUnitCost(forecast.item_sku)
        // Use any user-adjusted qty from a previous run, otherwise use forecasted_demand
        const desiredQty = adjustments.value[forecast.item_sku] ?? forecast.forecasted_demand
        const cost = desiredQty * unitCost

        if (cost <= remaining) {
          result.push({ ...forecast, unitCost, qty: desiredQty, partial: false })
          remaining -= cost
        } else {
          // Afford as many units as the remaining budget allows
          const affordableQty = Math.floor(remaining / unitCost)
          if (affordableQty > 0) {
            result.push({ ...forecast, unitCost, qty: affordableQty, partial: true })
            remaining = 0
          }
        }
      }

      recommendations.value = result
      hasCalculated.value = true
      orderMessage.value = ''
    }

    const totalCost = computed(() =>
      recommendations.value.reduce((sum, r) => {
        const qty = adjustments.value[r.item_sku] ?? r.qty
        return sum + qty * r.unitCost
      }, 0)
    )

    const remainingBudget = computed(() => budget.value - totalCost.value)

    const placeOrder = async () => {
      if (recommendations.value.length === 0 || totalCost.value > budget.value) return
      try {
        isSubmitting.value = true
        orderMessage.value = ''
        const items = recommendations.value.map(r => ({
          sku: r.item_sku,
          name: r.item_name,
          quantity: adjustments.value[r.item_sku] ?? r.qty,
          unit_cost: r.unitCost
        }))
        await api.createRestockingOrder({ items, lead_time_days: 14 })
        orderMessage.value = 'Restocking order placed successfully. View it in the Orders tab.'
        orderSuccess.value = true
        // Reset for next use
        recommendations.value = []
        adjustments.value = {}
        hasCalculated.value = false
      } catch (err) {
        orderMessage.value = 'Failed to place restocking order: ' + err.message
        orderSuccess.value = false
      } finally {
        isSubmitting.value = false
      }
    }

    onMounted(async () => {
      try {
        const [forecasts, inventory] = await Promise.all([
          api.getDemandForecasts(),
          api.getInventory({})
        ])
        allForecasts.value = forecasts
        // Build SKU → unit_cost lookup from inventory
        inventoryMap.value = Object.fromEntries(inventory.map(item => [item.sku, item.unit_cost]))
      } catch (err) {
        error.value = 'Failed to load data: ' + err.message
      } finally {
        loading.value = false
      }
    })

    return {
      t,
      loading,
      error,
      budget,
      recommendations,
      adjustments,
      hasCalculated,
      totalCost,
      remainingBudget,
      isSubmitting,
      orderMessage,
      orderSuccess,
      calculateRecommendations,
      placeOrder
    }
  }
}
</script>

<style scoped>
.restocking {
  padding: 0;
}

.page-header {
  margin-bottom: 24px;
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 4px;
}

.page-header p {
  color: #64748b;
  font-size: 0.9rem;
}

.card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 24px;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.card-title {
  font-size: 1rem;
  font-weight: 700;
  color: #0f172a;
}

/* Budget controls */
.budget-card {}

.budget-control {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.budget-display {
  font-size: 2rem;
  font-weight: 800;
  color: #0f172a;
  text-align: center;
}

.budget-slider {
  width: 100%;
  height: 6px;
  accent-color: #2563eb;
  cursor: pointer;
}

.budget-range-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  color: #94a3b8;
}

.budget-actions {
  display: flex;
  align-items: center;
  gap: 16px;
  flex-wrap: wrap;
}

.btn-calculate {
  background: #2563eb;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px 20px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-calculate:hover {
  background: #1d4ed8;
}

.priority-note {
  font-size: 0.78rem;
  color: #64748b;
  font-style: italic;
}

/* Stats grid */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 24px;
}

.stat-card {
  background: white;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 20px;
}

.stat-card-warning {
  border-color: #fca5a5;
  background: #fff5f5;
}

.stat-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 6px;
}

.stat-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
}

.stat-value.negative {
  color: #dc2626;
}

/* Table */
.table-container {
  overflow-x: auto;
}

.restocking-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.restocking-table th {
  text-align: left;
  padding: 10px 12px;
  font-size: 0.72rem;
  font-weight: 700;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  border-bottom: 2px solid #e2e8f0;
  background: #f8fafc;
}

.restocking-table td {
  padding: 12px;
  border-bottom: 1px solid #f1f5f9;
  color: #0f172a;
}

.restocking-table tbody tr:hover {
  background: #f8fafc;
}

.restocking-table tfoot td {
  padding: 12px;
  border-top: 2px solid #e2e8f0;
  background: #f8fafc;
  font-weight: 700;
}

.total-label {
  text-align: right;
  color: #64748b;
}

.total-value {
  color: #0f172a;
  font-size: 1rem;
}

/* Quantity input */
.qty-input {
  width: 80px;
  padding: 6px 8px;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  font-size: 0.875rem;
  color: #0f172a;
  text-align: center;
}

.qty-input:focus {
  outline: none;
  border-color: #2563eb;
}

/* Badges */
.badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 999px;
  font-size: 0.72rem;
  font-weight: 600;
}

.badge.increasing {
  background: #dcfce7;
  color: #166534;
}

.badge.stable {
  background: #dbeafe;
  color: #1e40af;
}

.partial-badge {
  display: inline-block;
  margin-left: 6px;
  padding: 1px 6px;
  border-radius: 4px;
  font-size: 0.65rem;
  font-weight: 600;
  background: #fef3c7;
  color: #92400e;
}

/* Order actions */
.order-actions {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-top: 20px;
  padding-top: 16px;
  border-top: 1px solid #f1f5f9;
  flex-wrap: wrap;
}

.btn-primary {
  background: #0f172a;
  color: white;
  border: none;
  border-radius: 8px;
  padding: 10px 24px;
  font-size: 0.9rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.btn-primary:hover:not(:disabled) {
  background: #1e293b;
}

.btn-primary:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.order-message {
  font-size: 0.85rem;
  font-weight: 500;
}

.order-message.success {
  color: #059669;
}

.order-message.error {
  color: #dc2626;
}

.order-message.warning {
  color: #d97706;
}

/* Empty state */
.empty-state {
  text-align: center;
  padding: 40px;
  color: #64748b;
  font-size: 0.9rem;
}

.loading {
  color: #64748b;
  padding: 40px;
  text-align: center;
}

.error {
  color: #dc2626;
  padding: 40px;
  text-align: center;
}
</style>
