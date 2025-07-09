<template>
  <div class="crud-container">
    <div class="header">
      <h1><i class="fas fa-chalkboard-teacher"></i> Sistema de Gestión de Profesores</h1>
      <p>
        Administra la información de los profesores, sus materias y documentación con esta
        herramienta intuitiva
      </p>
    </div>

    <section class="form-section">
      <div class="form-header">
        <h3 class="section-title">
          <i :class="isEditing ? 'fa-user-edit' : 'fa-user-plus'" class="fas"></i>
          {{ isEditing ? 'Editar Profesor' : 'Añadir Nuevo Profesor' }}
        </h3>
        <div v-if="isEditing" class="editing-badge">
          <i class="fas fa-pencil-alt"></i> Modo edición activado
        </div>
      </div>

      <div class="form-grid">
        <div class="form-group">
          <label><i class="fas fa-user"></i> Nombre:</label>
          <input
            v-model="teacher.teacherName"
            :class="{ 'input-error': errors.teacherName }"
            class="form-input"
            placeholder="Nombre del profesor"
            type="text"
          />
          <div v-if="errors.teacherName" class="error-message">{{ errors.teacherName }}</div>
        </div>

        <div class="form-group">
          <label><i class="fas fa-signature"></i> Apellidos:</label>
          <input
            v-model="teacher.surname"
            :class="{ 'input-error': errors.surname }"
            class="form-input"
            placeholder="Apellidos del profesor"
            type="text"
          />
          <div v-if="errors.surname" class="error-message">{{ errors.surname }}</div>
        </div>

        <div class="form-group">
          <label><i class="fas fa-id-card"></i> ID (DNI):</label>
          <input
            v-model="teacher.dni"
            :class="{ 'input-error': errors.dni }"
            :disabled="isEditing"
            class="form-input"
            placeholder="DNI/NIE del profesor"
            type="text"
          />
          <div v-if="errors.dni" class="error-message">{{ errors.dni }}</div>
        </div>

        <div class="form-group subjects-group">
          <label><i class="fas fa-book"></i> Materias:</label>
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
          <div v-if="errors.subjects" class="error-message">{{ errors.subjects }}</div>
        </div>

        <div class="form-group checkbox-group">
          <div class="toggle-switch">
            <input id="doc" v-model="teacher.doc" class="toggle-input" type="checkbox" />
            <label class="toggle-label" for="doc">
              <span class="toggle-handle"></span>
              <span class="toggle-text">
                <i
                  :class="teacher.doc ? 'fa-check-circle' : 'fa-exclamation-circle'"
                  class="fas"
                ></i>
                {{ teacher.doc ? 'Documentación entregada' : 'Documentación pendiente' }}
              </span>
            </label>
          </div>
        </div>
      </div>

      <div class="form-actions">
        <button v-if="!isEditing" class="btn primary-btn" @click="handleAddTeacher">
          <i class="fas fa-user-plus"></i> Agregar Profesor
        </button>
        <button v-else class="btn primary-btn" @click="handleUpdateTeacher">
          <i class="fas fa-save"></i> Guardar Cambios
        </button>
        <button class="btn secondary-btn" @click="cancelEditing">
          <i class="fas fa-times"></i> {{ isEditing ? 'Cancelar' : 'Limpiar' }}
        </button>
      </div>
    </section>

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
            @input="updateFilteredTeachers"
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
        <div v-if="localFilteredTeachers.length === 0" class="empty-list">
          <div class="empty-content">
            <i class="fas fa-search"></i>
            <h4>No se encontraron resultados</h4>
            <p>Intenta con otro término de búsqueda</p>
          </div>
        </div>

        <table v-else class="teachers-table">
          <thead>
            <tr>
              <th>Nombre</th>
              <th>Apellidos</th>
              <th>ID</th>
              <th>Materias</th>
              <th>Documentación</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="t in localFilteredTeachers" :key="t.dni">
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
                <button class="action-btn edit-btn" title="Editar" @click="startEditing(t)">
                  <i class="fas fa-edit"></i>
                </button>
                <button
                  class="action-btn delete-btn"
                  title="Eliminar"
                  @click="deleteTeacher(t.dni)"
                >
                  <i class="fas fa-trash"></i>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <div class="footer">
      <p>
        Sistema de Gestión de Profesores &copy; {{ new Date().getFullYear() }} | Desarrollado con
        Vue 3, TypeScript y buenas prácticas
      </p>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, watch } from 'vue' // Importa 'watch' para reaccionar a cambios en searchTerm

// Definición de interfaces
interface ITeacher {
  teacherName: string
  surname: string
  dni: string
  subjects: string[]
  doc: boolean
}

interface IErrors {
  teacherName?: string
  surname?: string
  dni?: string
  subjects?: string
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
const isEditing = ref(false)
const originalDni = ref('')
const searchTerm = ref('')
const errors = ref<IErrors>({})

// Variable para el resultado de la búsqueda
const localFilteredTeachers = ref<ITeacher[]>([])

// Función para actualizar los profesores filtrados (reemplaza computed)
const updateFilteredTeachers = () => {
  if (!searchTerm.value.trim()) {
    localFilteredTeachers.value = teachers.value
    return
  }

  const term = searchTerm.value.toLowerCase()
  localFilteredTeachers.value = teachers.value.filter(
    (t) =>
      t.teacherName.toLowerCase().includes(term) ||
      t.surname.toLowerCase().includes(term) ||
      t.dni.toLowerCase().includes(term) ||
      t.subjects.some((sub) => sub.toLowerCase().includes(term)),
  )
}

// Observar cambios en teachers y searchTerm para actualizar localFilteredTeachers
watch(
  teachers,
  () => {
    updateFilteredTeachers()
  },
  { deep: true, immediate: true },
) // 'deep' para detectar cambios dentro de los objetos en el array
// 'immediate' para que se ejecute la primera vez

// Agregar materia
const handleSubject = () => {
  const trimmedSubject = subject.value.trim()
  if (!trimmedSubject) return

  if (teacher.value.subjects.includes(trimmedSubject)) {
    alert('Esta materia ya está agregada')
    return
  }

  teacher.value.subjects.push(trimmedSubject)
  subject.value = ''
  errors.value.subjects = ''
}

// Eliminar materia
const removeSubject = (index: number) => {
  teacher.value.subjects.splice(index, 1)
}

// Validar profesor
const validateTeacher = (): boolean => {
  errors.value = {}
  let isValid = true

  if (!teacher.value.teacherName.trim()) {
    errors.value.teacherName = 'El nombre es obligatorio'
    isValid = false
  }

  if (!teacher.value.surname.trim()) {
    errors.value.surname = 'Los apellidos son obligatorios'
    isValid = false
  }

  if (!teacher.value.dni.trim()) {
    errors.value.dni = 'El DNI es obligatorio'
    isValid = false
  }

  if (teacher.value.subjects.length === 0) {
    errors.value.subjects = 'Debe agregar al menos una materia'
    isValid = false
  }

  // Verificar si el DNI ya existe (excepto en edición)
  if (!isEditing.value && teachers.value.some((t) => t.dni === teacher.value.dni)) {
    errors.value.dni = 'Este DNI ya está registrado'
    isValid = false
  }

  return isValid
}

// Agregar profesor
const handleAddTeacher = () => {
  if (!validateTeacher()) return

  teachers.value.push({ ...teacher.value })
  resetForm()
}

// Actualizar profesor
const handleUpdateTeacher = () => {
  if (!validateTeacher()) return

  const index = teachers.value.findIndex((t) => t.dni === originalDni.value)
  if (index !== -1) {
    // Actualiza el profesor en el array principal
    teachers.value[index] = { ...teacher.value } // Copia el objeto para mantener reactividad
  }

  resetForm()
}

// Cancelar edición
const cancelEditing = () => {
  resetForm()
}

// Eliminar profesor
const deleteTeacher = (dni: string) => {
  if (confirm('¿Está seguro de eliminar este profesor?')) {
    teachers.value = teachers.value.filter((t) => t.dni !== dni)
  }
}

// Iniciar edición
const startEditing = (selectedTeacher: ITeacher) => {
  teacher.value = { ...selectedTeacher } // Copia para evitar mutación directa
  originalDni.value = selectedTeacher.dni
  isEditing.value = true
  errors.value = {} // Limpiar errores al iniciar edición
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
  isEditing.value = false
  originalDni.value = ''
  errors.value = {}
}

// Datos iniciales para demostración (se ejecutan una vez al montar el componente)
teachers.value = [
  {
    teacherName: 'María',
    surname: 'García López',
    dni: '12345678A',
    subjects: ['Matemáticas', 'Física'],
    doc: true,
  },
  {
    teacherName: 'Carlos',
    surname: 'Martínez Ruiz',
    dni: '87654321B',
    subjects: ['Historia', 'Geografía'],
    doc: false,
  },
  {
    teacherName: 'Ana',
    surname: 'Fernández Sánchez',
    dni: '56781234C',
    subjects: ['Biología', 'Química'],
    doc: true,
  },
]

// Llama a la función de filtro al inicio para que localFilteredTeachers tenga los datos iniciales
updateFilteredTeachers()
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #f5f7fa 0%, #e4edf5 100%);
  color: #2d3748;
  line-height: 1.6;
  padding: 20px;
  min-height: 100vh;
}

.crud-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 30px;
}

.header {
  text-align: center;
  padding: 20px 0;
}

.header h1 {
  color: #2c3e50;
  font-size: 2.5rem;
  margin-bottom: 10px;
  position: relative;
  display: inline-block;
}

.header h1::after {
  content: '';
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 100px;
  height: 4px;
  background: #3498db;
  border-radius: 2px;
}

.header p {
  color: #4a5568;
  max-width: 700px;
  margin: 0 auto;
  font-size: 1.1rem;
}

.form-section,
.list-section {
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.08);
  padding: 30px;
  transition:
    transform 0.3s ease,
    box-shadow 0.3s ease;
}

.form-section:hover,
.list-section:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.12);
}

.section-title {
  color: #2c3e50;
  padding-bottom: 15px;
  margin-bottom: 25px;
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 1.6rem;
  font-weight: 600;
  border-bottom: 2px solid #3498db;
}

.section-title i {
  font-size: 1.4rem;
}

.form-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.editing-badge {
  background: #f39c12;
  color: white;
  padding: 8px 15px;
  border-radius: 20px;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 500;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 25px;
}

.form-group {
  margin-bottom: 25px;
}

.form-group label {
  display: block;
  margin-bottom: 10px;
  font-weight: 600;
  color: #2c3e50;
  font-size: 1rem;
}

.form-input {
  width: 100%;
  padding: 14px 18px;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  font-size: 1rem;
  transition: all 0.3s;
  background: #f8fafc;
}

.form-input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 4px rgba(52, 152, 219, 0.2);
  background: white;
}

.form-input:disabled {
  background: #edf2f7;
  color: #718096;
}

.subjects-group {
  grid-column: 1 / -1;
}

.subject-input-group {
  display: flex;
  gap: 12px;
  margin-bottom: 15px;
}

.subject-input-group input {
  flex: 1;
}

.subject-list {
  background: #f8fafc;
  border-radius: 10px;
  padding: 18px;
  min-height: 60px;
  border: 1px dashed #cbd5e0;
}

.empty-subjects {
  color: #718096;
  font-style: italic;
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 0.95rem;
}

.subject-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.subject-tag {
  background: #e3f2fd;
  color: #1976d2;
  padding: 8px 15px;
  border-radius: 20px;
  font-size: 0.95rem;
  display: flex;
  align-items: center;
  gap: 8px;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.05);
  transition: all 0.2s;
}

.subject-tag:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 8px rgba(0, 0, 0, 0.08);
}

.remove-icon {
  cursor: pointer;
  font-size: 0.9rem;
  opacity: 0.7;
  transition: all 0.2s;
}

.remove-icon:hover {
  opacity: 1;
  color: #e74c3c;
}

.checkbox-group {
  display: flex;
  align-items: center;
  gap: 15px;
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
  width: 55px;
  height: 28px;
  background: #e2e8f0;
  border-radius: 14px;
  position: relative;
  margin-right: 12px;
  transition: background 0.3s;
}

.toggle-handle:after {
  content: '';
  position: absolute;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: white;
  top: 2px;
  left: 2px;
  transition: transform 0.3s;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.toggle-input:checked + .toggle-label .toggle-handle {
  background: #3498db;
}

.toggle-input:checked + .toggle-label .toggle-handle:after {
  transform: translateX(27px);
}

.toggle-text {
  font-weight: 500;
  font-size: 1rem;
}

.form-actions {
  display: flex;
  gap: 15px;
  margin-top: 25px;
  flex-wrap: wrap;
  justify-content: center;
}

.btn {
  padding: 12px 25px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  font-weight: 600;
  display: inline-flex;
  align-items: center;
  gap: 10px;
  transition: all 0.3s;
  font-size: 1rem;
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.primary-btn {
  background: #3498db;
  color: white;
  box-shadow: 0 4px 6px rgba(52, 152, 219, 0.3);
}

.primary-btn:hover:not(:disabled) {
  background: #2980b9;
  transform: translateY(-3px);
  box-shadow: 0 6px 10px rgba(52, 152, 219, 0.4);
}

.secondary-btn {
  background: #95a5a6;
  color: white;
  box-shadow: 0 4px 6px rgba(149, 165, 166, 0.3);
}

.secondary-btn:hover {
  background: #7f8c8d;
  transform: translateY(-3px);
  box-shadow: 0 6px 10px rgba(149, 165, 166, 0.4);
}

.add-btn {
  background: #2ecc71;
  color: white;
  padding: 12px 18px;
  white-space: nowrap;
  box-shadow: 0 4px 6px rgba(46, 204, 113, 0.3);
}

.add-btn:hover {
  background: #27ae60;
  transform: translateY(-3px);
  box-shadow: 0 6px 10px rgba(46, 204, 113, 0.4);
}

.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 25px;
}

.count-badge {
  background: #3498db;
  color: white;
  padding: 4px 12px;
  border-radius: 15px;
  font-size: 1rem;
  margin-left: 10px;
}

.search-box {
  position: relative;
  min-width: 280px;
  flex: 1;
  max-width: 400px;
}

.search-input {
  padding: 12px 15px 12px 45px;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  width: 100%;
  background: #f8fafc;
  transition: all 0.3s;
  font-size: 1rem;
}

.search-input:focus {
  border-color: #3498db;
  outline: none;
  box-shadow: 0 0 0 4px rgba(52, 152, 219, 0.2);
  background: white;
}

.search-icon {
  position: absolute;
  left: 18px;
  top: 50%;
  transform: translateY(-50%);
  color: #a0aec0;
  font-size: 1.1rem;
}

.empty-list {
  text-align: center;
  padding: 50px 20px;
  color: #718096;
  background: #f8fafc;
  border-radius: 12px;
  border: 2px dashed #e2e8f0;
}

.empty-content {
  max-width: 450px;
  margin: 0 auto;
}

.empty-content i {
  font-size: 3.5rem;
  color: #cbd5e0;
  margin-bottom: 20px;
}

.empty-content h4 {
  font-size: 1.5rem;
  margin-bottom: 15px;
  color: #4a5568;
}

.empty-content p {
  font-size: 1.1rem;
}

.teachers-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.06);
}

.teachers-table th,
.teachers-table td {
  padding: 16px 18px;
  text-align: left;
  border-bottom: 1px solid #edf2f7;
}

.teachers-table th {
  background: #3498db;
  color: white;
  font-weight: 600;
  font-size: 1.05rem;
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
  gap: 8px;
}

.status-pending {
  color: #e74c3c;
  font-weight: 500;
  display: flex;
  align-items: center;
  gap: 8px;
}

.actions-cell {
  display: flex;
  gap: 10px;
}

.action-btn {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
}

.action-btn:hover {
  transform: scale(1.1);
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.15);
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

.footer {
  text-align: center;
  padding: 20px;
  color: #718096;
  margin-top: 20px;
  font-size: 0.95rem;
}

.footer a {
  color: #3498db;
  text-decoration: none;
}

.footer a:hover {
  text-decoration: underline;
}

@media (max-width: 768px) {
  .form-grid {
    grid-template-columns: 1fr;
  }

  .form-header,
  .list-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 20px;
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
    width: 36px;
    height: 36px;
    font-size: 0.9rem;
  }
}

@media (max-width: 480px) {
  .header h1 {
    font-size: 2rem;
  }

  .section-title {
    font-size: 1.4rem;
  }

  .form-input,
  .search-input {
    padding: 12px 15px;
  }

  .btn {
    padding: 10px 18px;
    font-size: 0.95rem;
  }
}
</style>
