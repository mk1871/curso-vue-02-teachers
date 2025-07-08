<script lang="ts" setup>
import { computed, ref } from 'vue'

interface ITeacher {
  teacherName: string
  surname: string
  dni: string
  subjects: string[]
  doc: boolean
}

// Estado inicial
const teacher = ref<ITeacher>({
  teacherName: '',
  surname: '',
  dni: '',
  subjects: [],
  doc: false,
})

const teachers = ref<ITeacher[]>([])
const subject = ref('')
const editing = ref(false)
const editIndex = ref<number | null>(null)
const searchTerm = ref('')

// Agregar materia
const handleSubject = () => {
  if (subject.value.trim() && !teacher.value.subjects.includes(subject.value.trim())) {
    teacher.value.subjects.push(subject.value.trim())
    subject.value = ''
  }
}

// Eliminar materia
const removeSubject = (index: number) => {
  teacher.value.subjects.splice(index, 1)
}

// Validar profesor
const validateTeacher = (): boolean => {
  if (!teacher.value.teacherName.trim()) {
    alert('El nombre es obligatorio')
    return false
  }
  if (!teacher.value.surname.trim()) {
    alert('Los apellidos son obligatorios')
    return false
  }
  if (!teacher.value.dni.trim()) {
    alert('El DNI es obligatorio')
    return false
  }
  if (teacher.value.subjects.length === 0) {
    alert('Debe agregar al menos una materia')
    return false
  }

  // Verificar si el DNI ya existe (excepto en edición)
  if (!editing.value && teachers.value.some((t) => t.dni === teacher.value.dni)) {
    alert('Este DNI ya está registrado')
    return false
  }

  return true
}

// Agregar profesor
const handleAddTeacher = () => {
  if (!validateTeacher()) return

  teachers.value.push({ ...teacher.value })
  resetForm()
}

// Editar profesor
const editTeacher = (index: number) => {
  teacher.value = { ...teachers.value[index] }
  editing.value = true
  editIndex.value = index
}

// Actualizar profesor
const updateTeacher = () => {
  if (editIndex.value === null || !validateTeacher()) return

  teachers.value[editIndex.value] = { ...teacher.value }
  resetForm()
}

// Eliminar profesor
const deleteTeacher = (index: number) => {
  if (confirm('¿Está seguro de eliminar este profesor?')) {
    teachers.value.splice(index, 1)
  }
}

// Reiniciar formulario
const resetForm = () => {
  teacher.value = {
    teacherName: '',
    surname: '',
    dni: '',
    subjects: [],
    doc: false,
  }
  subject.value = ''
  editing.value = false
  editIndex.value = null
}

// Filtrar profesores
const filteredTeachers = computed(() => {
  if (!searchTerm.value) return teachers.value

  const term = searchTerm.value.toLowerCase()
  return teachers.value.filter(
    (t) =>
      t.teacherName.toLowerCase().includes(term) ||
      t.surname.toLowerCase().includes(term) ||
      t.dni.toLowerCase().includes(term) ||
      t.subjects.some((sub) => sub.toLowerCase().includes(term)),
  )
})
</script>

<template>
  <div class="crud-container">
    <!-- Formulario para agregar/editar profesores -->
    <section class="form-section">
      <div class="form-header">
        <h3 class="section-title">
          <i :class="editing ? 'fa-user-edit' : 'fa-user-plus'" class="fas"></i>
          {{ editing ? 'Editar Profesor' : 'Añadir Nuevo Profesor' }}
        </h3>
        <div v-if="editing" class="editing-badge">
          <i class="fas fa-pencil-alt"></i> Modo edición
        </div>
      </div>

      <div class="form-grid">
        <div class="form-group">
          <label>Nombre:</label>
          <input
            v-model="teacher.teacherName"
            class="form-input"
            placeholder="Nombre del profesor"
            type="text"
          />
        </div>

        <div class="form-group">
          <label>Apellidos:</label>
          <input
            v-model="teacher.surname"
            class="form-input"
            placeholder="Apellidos del profesor"
            type="text"
          />
        </div>

        <div class="form-group">
          <label>ID (DNI):</label>
          <input
            v-model="teacher.dni"
            :disabled="editing"
            class="form-input"
            placeholder="DNI/NIE del profesor"
            type="text"
          />
        </div>

        <div class="form-group subjects-group">
          <label>Materias:</label>
          <div class="subject-input-group">
            <input
              v-model="subject"
              class="form-input"
              placeholder="Ingrese una materia"
              type="text"
              @keyup.enter="handleSubject"
            />
            <button :disabled="!subject.trim()" class="btn add-btn" @click="handleSubject">
              <i class="fas fa-plus"></i> Agregar
            </button>
          </div>

          <div class="subject-list">
            <div v-if="teacher.subjects.length === 0" class="empty-subjects">
              <i class="fas fa-book"></i> No se han agregado materias
            </div>
            <div v-else class="subject-tags">
              <div v-for="(sub, index) in teacher.subjects" :key="index" class="subject-tag">
                {{ sub }}
                <i class="fas fa-times remove-icon" @click="removeSubject(index)"></i>
              </div>
            </div>
          </div>
        </div>

        <div class="form-group checkbox-group">
          <div class="toggle-switch">
            <input id="doc" v-model="teacher.doc" class="toggle-input" type="checkbox" />
            <label class="toggle-label" for="doc">
              <span class="toggle-handle"></span>
              <span class="toggle-text">{{
                teacher.doc ? 'Documentación entregada' : 'Documentación pendiente'
              }}</span>
            </label>
          </div>
        </div>
      </div>

      <div class="form-actions">
        <button
          v-if="!editing"
          :disabled="!teacher.teacherName || !teacher.surname || !teacher.dni"
          class="btn primary-btn"
          @click="handleAddTeacher"
        >
          <i class="fas fa-user-plus"></i> Agregar Profesor
        </button>
        <button
          v-else
          :disabled="!teacher.teacherName || !teacher.surname || !teacher.dni"
          class="btn primary-btn"
          @click="updateTeacher"
        >
          <i class="fas fa-save"></i> Guardar Cambios
        </button>
        <button class="btn secondary-btn" @click="resetForm">
          <i class="fas fa-times"></i> {{ editing ? 'Cancelar' : 'Limpiar' }}
        </button>
      </div>
    </section>

    <!-- Listado de profesores -->
    <section class="list-section">
      <div class="list-header">
        <h3 class="section-title">
          <i class="fas fa-list"></i> Listado de Profesores
          <span class="count-badge">{{ teachers.length }}</span>
        </h3>

        <div class="search-box">
          <i class="fas fa-search search-icon"></i>
          <input
            v-model="searchTerm"
            class="search-input"
            placeholder="Buscar profesores..."
            type="text"
          />
        </div>
      </div>

      <div v-if="teachers.length === 0" class="empty-list">
        <div class="empty-content">
          <i class="fas fa-user-graduate"></i>
          <h4>No hay profesores registrados</h4>
          <p>Comienza agregando un nuevo profesor usando el formulario</p>
        </div>
      </div>

      <div v-else>
        <div v-if="filteredTeachers.length === 0" class="empty-list">
          <div class="empty-content">
            <i class="fas fa-search"></i>
            <h4>No se encontraron resultados</h4>
            <p>Intenta con otro término de búsqueda</p>
          </div>
        </div>

        <table v-else class="teachers-table">
          <thead>
            <tr>
              <th @click="sortBy('teacherName')">Nombre <i class="fas fa-sort"></i></th>
              <th @click="sortBy('surname')">Apellidos <i class="fas fa-sort"></i></th>
              <th>ID</th>
              <th>Materias</th>
              <th>Documentación</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(t, index) in filteredTeachers" :key="index">
              <td>{{ t.teacherName }}</td>
              <td>{{ t.surname }}</td>
              <td>{{ t.dni }}</td>
              <td>
                <div class="subject-tags">
                  <span v-for="(sub, subIndex) in t.subjects" :key="subIndex" class="subject-tag">
                    {{ sub }}
                  </span>
                </div>
              </td>
              <td>
                <span :class="t.doc ? 'status-delivered' : 'status-pending'">
                  <i :class="t.doc ? 'fa-check-circle' : 'fa-exclamation-circle'" class="fas"></i>
                  {{ t.doc ? 'Entregada' : 'Pendiente' }}
                </span>
              </td>
              <td class="actions-cell">
                <button class="action-btn edit-btn" title="Editar" @click="editTeacher(index)">
                  <i class="fas fa-edit"></i>
                </button>
                <button
                  class="action-btn delete-btn"
                  title="Eliminar"
                  @click="deleteTeacher(index)"
                >
                  <i class="fas fa-trash"></i>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>
  </div>
</template>

<style scoped>
.crud-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #2d3748;
}

.section-title {
  color: #2c3e50;
  padding-bottom: 12px;
  margin-bottom: 20px;
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 1.4rem;
  font-weight: 600;
  border-bottom: 2px solid #3498db;
}

.form-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.editing-badge {
  background: #f39c12;
  color: white;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.85rem;
  display: flex;
  align-items: center;
  gap: 6px;
}

.form-section {
  background: white;
  border-radius: 12px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
  padding: 25px;
  margin-bottom: 30px;
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
}

.form-section:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.12);
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #2c3e50;
  font-size: 0.95rem;
}

.form-input {
  width: 100%;
  padding: 12px 15px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1rem;
  transition: all 0.3s;
  background: #f8fafc;
}

.form-input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
  background: white;
}

.form-input:disabled {
  background: #edf2f7;
  color: #718096;
}

.subject-input-group {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.subject-input-group input {
  flex: 1;
}

.subject-list {
  background: #f8fafc;
  border-radius: 8px;
  padding: 15px;
  min-height: 50px;
  border: 1px dashed #cbd5e0;
}

.empty-subjects {
  color: #718096;
  font-style: italic;
  display: flex;
  align-items: center;
  gap: 8px;
}

.subject-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.subject-tag {
  background: #e3f2fd;
  color: #1976d2;
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 6px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: all 0.2s;
}

.subject-tag:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.08);
}

.remove-icon {
  cursor: pointer;
  font-size: 0.8rem;
  opacity: 0.7;
  transition: all 0.2s;
}

.remove-icon:hover {
  opacity: 1;
  color: #e74c3c;
}

.toggle-switch {
  position: relative;
  display: inline-block;
  width: auto;
}

.toggle-input {
  display: none;
}

.toggle-label {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.toggle-handle {
  display: inline-block;
  width: 50px;
  height: 26px;
  background: #e2e8f0;
  border-radius: 13px;
  position: relative;
  margin-right: 10px;
  transition: background 0.3s;
}

.toggle-handle:after {
  content: '';
  position: absolute;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: white;
  top: 2px;
  left: 2px;
  transition: transform 0.3s;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.toggle-input:checked + .toggle-label .toggle-handle {
  background: #3498db;
}

.toggle-input:checked + .toggle-label .toggle-handle:after {
  transform: translateX(24px);
}

.toggle-text {
  font-weight: 500;
}

.form-actions {
  display: flex;
  gap: 12px;
  margin-top: 20px;
  flex-wrap: wrap;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  transition: all 0.3s;
  font-size: 0.95rem;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.primary-btn {
  background: #3498db;
  color: white;
}

.primary-btn:hover:not(:disabled) {
  background: #2980b9;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(52, 152, 219, 0.3);
}

.secondary-btn {
  background: #95a5a6;
  color: white;
}

.secondary-btn:hover {
  background: #7f8c8d;
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(149, 165, 166, 0.3);
}

.add-btn {
  background: #2ecc71;
  color: white;
  padding: 10px 15px;
  white-space: nowrap;
}

.add-btn:hover {
  background: #27ae60;
}

.list-section {
  background: white;
  border-radius: 12px;
  box-shadow: 0 5px 20px rgba(0, 0, 0, 0.08);
  padding: 25px;
  transition: transform 0.3s ease;
}

.list-section:hover {
  transform: translateY(-2px);
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 15px;
  margin-bottom: 20px;
}

.count-badge {
  background: #3498db;
  color: white;
  padding: 2px 10px;
  border-radius: 12px;
  font-size: 0.9rem;
  margin-left: 8px;
}

.search-box {
  position: relative;
  min-width: 250px;
}

.search-input {
  padding: 10px 15px 10px 40px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  width: 100%;
  background: #f8fafc;
  transition: all 0.3s;
}

.search-input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.2);
  background: white;
}

.search-icon {
  position: absolute;
  left: 15px;
  top: 50%;
  transform: translateY(-50%);
  color: #a0aec0;
}

.empty-list {
  text-align: center;
  padding: 40px 20px;
  color: #718096;
  background: #f8fafc;
  border-radius: 10px;
  border: 2px dashed #e2e8f0;
}

.empty-content {
  max-width: 400px;
  margin: 0 auto;
}

.empty-content i {
  font-size: 3rem;
  color: #cbd5e0;
  margin-bottom: 15px;
}

.empty-content h4 {
  font-size: 1.3rem;
  margin-bottom: 10px;
  color: #4a5568;
}

.empty-content p {
  font-size: 1rem;
}

.teachers-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.teachers-table th,
.teachers-table td {
  padding: 14px 16px;
  text-align: left;
  border-bottom: 1px solid #edf2f7;
}

.teachers-table th {
  background: #3498db;
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.3s;
}

.teachers-table th:hover {
  background: #2980b9;
}

.teachers-table tr:nth-child(even) {
  background: #f8fafc;
}

.teachers-table tr:hover {
  background: #ebf4ff;
}

.status-delivered {
  color: #27ae60;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 6px;
}

.status-pending {
  color: #e74c3c;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 6px;
}

.actions-cell {
  display: flex;
  gap: 8px;
}

.action-btn {
  width: 36px;
  height: 36px;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  cursor: pointer;
  transition: all 0.3s;
}

.action-btn:hover {
  transform: scale(1.1);
}

.edit-btn {
  background: #f39c12;
  color: white;
}

.edit-btn:hover {
  background: #d35400;
}

.delete-btn {
  background: #e74c3c;
  color: white;
}

.delete-btn:hover {
  background: #c0392b;
}

@media (max-width: 768px) {
  .form-grid {
    grid-template-columns: 1fr;
  }

  .form-header,
  .list-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }

  .subject-input-group {
    flex-direction: column;
  }

  .form-actions {
    flex-direction: column;
  }

  .teachers-table {
    display: block;
    overflow-x: auto;
  }

  .action-btn {
    width: 32px;
    height: 32px;
    font-size: 0.9rem;
  }
}
</style>
