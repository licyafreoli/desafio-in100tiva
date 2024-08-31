# desafio-in100tiva

document.addEventListener('DOMContentLoaded', () => {
    const state = {
        classes: [
            { id: 1, name: "Matemática - 9º Ano", students: [], late: 0, difficulties: 0, lessons: 12, lessonDetails: [] },
            { id: 2, name: "Português - 10º Ano", students: [], late: 0, difficulties: 0, lessons: 8, lessonDetails: [] },
            { id: 3, name: "Ciências - 8º Ano", students: [], late: 0, difficulties: 0, lessons: 18, lessonDetails: [] }
        ],
        currentClass: null,
        currentLesson: null
    };

    const elements = {
        classList: document.getElementById('classList'),
        currentClassElement: document.getElementById('currentClass'),
        studentCountElement: document.getElementById('studentCount'),
        lateCountElement: document.getElementById('lateCount'),
        difficultyCountElement: document.getElementById('difficultyCount'),
        classLessonsElement: document.getElementById('classLessons'),
        addStudentBtn: document.getElementById('addStudentBtn'),
        studentList: document.getElementById('studentList'),
        lessonList: document.getElementById('lessonList'),
        addClassModal: document.getElementById('addClassModal'),
        addStudentModal: document.getElementById('addStudentModal'),
        editLessonModal: document.getElementById('editLessonModal')
    };

    function renderClasses() {
        elements.classList.innerHTML = state.classes.map(cls => `
            <li class="class-item ${state.currentClass && state.currentClass.id === cls.id ? 'selected' : ''}" data-class-id="${cls.id}">
                <div class="class-info">
                    <strong>${cls.name}</strong><br>
                    ${cls.students.length} alunos | ${cls.lessons} aulas
                </div>
                <div class="class-actions">
                    <span class="expand-icon">↔</span>
                    <span class="delete-icon" title="Excluir turma">🗑️</span>
                </div>
            </li>
        `).join('');

        attachClassListeners();
    }

    function attachClassListeners() {
        elements.classList.querySelectorAll('.class-item').forEach(item => {
            item.querySelector('.class-info').addEventListener('click', () => {
                state.currentClass = state.classes.find(cls => cls.id === parseInt(item.dataset.classId));
                renderClasses();
                updateClassDetails();
            });

            item.querySelector('.delete-icon').addEventListener('click', (e) => {
                e.stopPropagation();
                deleteClass(parseInt(item.dataset.classId));
            });
        });
    }

    function deleteClass(classId) {
        if (confirm('Tem certeza que deseja excluir esta turma?')) {
            state.classes = state.classes.filter(cls => cls.id !== classId);
            if (state.currentClass && state.currentClass.id === classId) {
                state.currentClass = null;
            }
            renderClasses();
            updateClassDetails();
        }
    }

    function updateClassDetails() {
        if (state.currentClass) {
            elements.currentClassElement.textContent = state.currentClass.name;
            elements.studentCountElement.innerHTML = `<div class="stat-icon"></div> ${state.currentClass.students.length} alunos`;
            elements.lateCountElement.innerHTML = `<div class="stat-icon"></div> ${state.currentClass.late} atrasados`;
            elements.difficultyCountElement.innerHTML = `<div class="stat-icon"></div> ${state.currentClass.difficulties} dificuldades`;
            elements.classLessonsElement.innerHTML = `<div class="stat-icon"></div> ${state.currentClass.lessons} aulas`;
            elements.addStudentBtn.style.display = 'block';
            renderStudents();
            renderLessons();
        } else {
            elements.currentClassElement.textContent = 'Selecione uma turma para ver os detalhes';
            elements.studentCountElement.innerHTML = '';
            elements.lateCountElement.innerHTML = '';
            elements.difficultyCountElement.innerHTML = '';
            elements.classLessonsElement.innerHTML = '';
            elements.addStudentBtn.style.display = 'none';
            elements.studentList.innerHTML = '';
            elements.lessonList.innerHTML = '';
        }
    }

    function renderStudents() {
        if (state.currentClass) {
            elements.studentList.innerHTML = state.currentClass.students.map(student => `
                <li class="student-item">
                    <div>
                        <strong>${student.name}</strong><br>
                        ${student.issue || 'Nenhum problema relatado'}
                    </div>
                    <span class="expand-icon">↔</span>
                </li>
            `).join('');
        } else {
            elements.studentList.innerHTML = '';
        }
    }

    function renderLessons() {
        if (state.currentClass) {
            elements.lessonList.innerHTML = Array.from({length: state.currentClass.lessons}, (_, i) => {
                const lessonDetail = state.currentClass.lessonDetails[i] || { title: `Aula ${i + 1}`, observation: '' };
                return `
                    <li class="lesson-item" data-lesson="${i}">
                        <div>
                            <strong>${lessonDetail.title}</strong><br>
                            ${lessonDetail.observation ? 'Obs: ' + lessonDetail.observation : 'Sem observações'}
                        </div>
                        <span class="expand-icon">↔</span>
                    </li>
                `;
            }).join('');

            attachLessonListeners();
        } else {
            elements.lessonList.innerHTML = '';
        }
    }

    function attachLessonListeners() {
        elements.lessonList.querySelectorAll('.lesson-item').forEach(item => {
            item.addEventListener('click', () => {
                state.currentLesson = parseInt(item.dataset.lesson);
                openEditLessonModal();
            });
        });
    }

    function openModal(modal) {
        modal.style.display = "block";
    }

    function closeModal(modal) {
        modal.style.display = "none";
    }

    function openEditLessonModal() {
        if (state.currentClass && state.currentLesson !== null) {
            const lessonDetail = state.currentClass.lessonDetails[state.currentLesson] || { title: `Aula ${state.currentLesson + 1}`, observation: '' };
            document.getElementById('lessonTitle').value = lessonDetail.title;
            document.getElementById('lessonObservation').value = lessonDetail.observation;
            openModal(elements.editLessonModal);
        }
    }

    document.getElementById('addClassForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const name = document.getElementById('className').value;
        const lessons = parseInt(document.getElementById('classLessons').value);
        const newClass = { 
            id: Date.now(), 
            name, 
            students: [], 
            late: 0, 
            difficulties: 0, 
            lessons, 
            lessonDetails: Array.from({length: lessons}, (_, i) => ({ title: `Aula ${i + 1}`, observation: '' }))
        };
        state.classes.push(newClass);
        state.currentClass = newClass;
        renderClasses();
        updateClassDetails();
        closeModal(elements.addClassModal);
        this.reset();
        
        // Forçar a exibição da aba de aulas
        const lessonsTab = document.querySelector('.tab[data-tab="lessons"]');
        if (lessonsTab) {
            lessonsTab.click();
        }
    });

    document.getElementById('addStudentForm').addEventListener('submit', function(e) {
        e.preventDefault();
        if (!state.currentClass) return;
        const name = document.getElementById('studentName').value;
        const issue = document.getElementById('studentIssue').value;
        state.currentClass.students.push({ id: Date.now(), name, issue });
        renderClasses();
        updateClassDetails();
        closeModal(elements.addStudentModal);
        this.reset();
    });

    document.getElementById('editLessonForm').addEventListener('submit', function(e) {
        e.preventDefault();
        if (!state.currentClass || state.currentLesson === null) return;
        const title = document.getElementById('lessonTitle').value;
        const observation = document.getElementById('lessonObservation').value;
        state.currentClass.lessonDetails[state.currentLesson] = { title, observation };
        renderLessons();
        closeModal(elements.editLessonModal);
        this.reset();
    });

    document.getElementById('newClassBtn').addEventListener('click', () => openModal(elements.addClassModal));
    elements.addStudentBtn.addEventListener('click', () => {
        if (state.currentClass) {
            openModal(elements.addStudentModal);
        }
    });

    document.querySelectorAll('.close').forEach(closeBtn => {
        closeBtn.addEventListener('click', () => closeModal(closeBtn.closest('.modal')));
    });

    document.querySelectorAll('.tab').forEach(tab => {
        tab.addEventListener('click', () => {
            document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
            document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
            tab.classList.add('active');
            document.getElementById(`${tab.dataset.tab}Tab`).classList.add('active');
        });
    });

    renderClasses();
    updateClass
