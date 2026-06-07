# LLM Output

=== FILE: src/App.jsx ===
import React, { useState, useEffect, useCallback, useRef } from 'react';
import { HashRouter, Routes, Route, Link } from 'react-router-dom';
import axios from 'axios';
import { ToastContainer } from 'react-toastify';
import { toast } from 'react-toastify';
import 'react-toastify/dist/ReactToastify.css';
import { format } from 'date-fns';
import { useForm } from 'react-hook-form';
import { clsx } from 'clsx';
import { FiCalendar, FiClock, FiUser, FiEdit, FiTrash } from 'react-icons/fi';

const BASE_URL = import.meta.env.VITE_API_URL || "http://localhost:8000";

function Header() {
  return (
    <header className="bg-blue-500 text-white p-4 flex justify-between">
      <h1 className="text-2xl font-bold">Hospital Management System</h1>
      <nav>
        <ul className="flex space-x-4">
          <li>
            <Link to="/" className="hover:text-gray-200">Appointments</Link>
          </li>
          <li>
            <Link to="/patients" className="hover:text-gray-200">Patients</Link>
          </li>
          <li>
            <Link to="/doctors" className="hover:text-gray-200">Doctors</Link>
          </li>
        </ul>
      </nav>
    </header>
  );
}

function Footer() {
  return (
    <footer className="bg-blue-500 text-white p-4 text-center">
      &copy; 2024 Hospital Management System
    </footer>
  );
}

function AppointmentList({ appointments, filteredAppointments, handleFilter }) {
  const [department, setDepartment] = useState('All');

  const handleDepartmentChange = (e) => {
    setDepartment(e.target.value);
    handleFilter(e.target.value);
  };

  return (
    <div>
      <h2 className="text-2xl font-bold mb-4">Appointments</h2>
      <select
        value={department}
        onChange={handleDepartmentChange}
        className="bg-white border border-gray-300 rounded py-2 px-4 mb-4"
      >
        <option value="All">All</option>
        <option value="General">General</option>
        <option value="Cardiology">Cardiology</option>
        <option value="Neurology">Neurology</option>
        <option value="Orthopedics">Orthopedics</option>
        <option value="Pediatrics">Pediatrics</option>
      </select>
      <table className="w-full table-auto border border-gray-300">
        <thead className="bg-gray-100">
          <tr>
            <th className="py-2 px-4">Doctor</th>
            <th className="py-2 px-4">Department</th>
            <th className="py-2 px-4">Date/Time</th>
            <th className="py-2 px-4">Status</th>
          </tr>
        </thead>
        <tbody>
          {filteredAppointments.map((appointment) => (
            <tr key={appointment.id}>
              <td className="py-2 px-4">{appointment.doctor}</td>
              <td className="py-2 px-4">{appointment.department}</td>
              <td className="py-2 px-4">
                {format(new Date(appointment.date), 'yyyy-MM-dd')}{' '}
                {format(new Date(appointment.time), 'hh:mm a')}
              </td>
              <td className="py-2 px-4">
                <span
                  className={clsx(
                    'py-1 px-2 rounded',
                    appointment.status === 'Scheduled'
                      ? 'bg-green-200 text-green-600'
                      : appointment.status === 'Pending'
                      ? 'bg-yellow-200 text-yellow-600'
                      : 'bg-red-200 text-red-600'
                  )}
                >
                  {appointment.status}
                </span>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}

function BookAppointmentForm({ handleSubmit }) {
  const {
    register,
    handleSubmit: handleFormSubmit,
    formState: { errors },
  } = useForm();

  return (
    <form onSubmit={handleFormSubmit(handleSubmit)} className="space-y-4">
      <div>
        <label className="block text-gray-700">Patient Name</label>
        <input
          type="text"
          {...register('patientName', { required: true })}
          className="bg-white border border-gray-300 rounded py-2 px-4"
        />
        {errors.patientName && (
          <p className="text-red-500">Patient name is required</p>
        )}
      </div>
      <div>
        <label className="block text-gray-700">Doctor</label>
        <input
          type="text"
          {...register('doctor', { required: true })}
          className="bg-white border border-gray-300 rounded py-2 px-4"
        />
        {errors.doctor && <p className="text-red-500">Doctor is required</p>}
      </div>
      <div>
        <label className="block text-gray-700">Department</label>
        <select
          {...register('department', { required: true })}
          className="bg-white border border-gray-300 rounded py-2 px-4"
        >
          <option value="">Select Department</option>
          <option value="General">General</option>
          <option value="Cardiology">Cardiology</option>
          <option value="Neurology">Neurology</option>
          <option value="Orthopedics">Orthopedics</option>
          <option value="Pediatrics">Pediatrics</option>
        </select>
        {errors.department && (
          <p className="text-red-500">Department is required</p>
        )}
      </div>
      <div>
        <label className="block text-gray-700">Date</label>
        <input
          type="date"
          {...register('date', { required: true })}
          className="bg-white border border-gray-300 rounded py-2 px-4"
        />
        {errors.date && <p className="text-red-500">Date is required</p>}
      </div>
      <div>
        <label className="block text-gray-700">Time</label>
        <input
          type="time"
          {...register('time', { required: true })}
          className="bg-white border border-gray-300 rounded py-2 px-4"
        />
        {errors.time && <p className="text-red-500">Time is required</p>}
      </div>
      <div>
        <label className="block text-gray-700">Notes</label>
        <textarea
          {...register('notes')}
          className="bg-white border border-gray-300 rounded py-2 px-4"
        />
      </div>
      <button
        type="submit"
        className="bg-blue-500 text-white py-2 px-4 rounded"
      >
        Book Appointment
      </button>
    </form>
  );
}

function App() {
  const [appointments, setAppointments] = useState([]);
  const [filteredAppointments, setFilteredAppointments] = useState([]);
  const [department, setDepartment] = useState('All');

  const handleFilter = (department) => {
    if (department === 'All') {
      setFilteredAppointments(appointments);
    } else {
      setFilteredAppointments(
        appointments.filter((appointment) => appointment.department === department)
      );
    }
  };

  const handleSubmit = async (data) => {
    try {
      const response = await axios.post(`${BASE_URL}/appointments`, data);
      toast.success('Appointment booked successfully');
      setAppointments((prevAppointments) => [...prevAppointments, response.data]);
    } catch (error) {
      toast.error('Error booking appointment');
    }
  };

  useEffect(() => {
    const fetchAppointments = async () => {
      try {
        const response = await axios.get(`${BASE_URL}/appointments`);
        const safeList = Array.isArray(response.data)
          ? response.data
          : (response.data?.items || []);
        setAppointments(safeList);
        setFilteredAppointments(safeList);
      } catch (error) {
        toast.error('Error fetching appointments');
      }
    };
    fetchAppointments();
  }, []);

  return (
    <HashRouter>
      <Header />
      <main className="container mx-auto p-4">
        <Routes>
          <Route
            path="/"
            element={
              <div>
                <h2 className="text-2xl font-bold mb-4">Appointments</h2>
                <AppointmentList
                  appointments={appointments}
                  filteredAppointments={filteredAppointments}
                  handleFilter={handleFilter}
                />
                <BookAppointmentForm handleSubmit={handleSubmit} />
              </div>
            }
          />
          <Route
            path="/patients"
            element={
              <div>
                <h2 className="text-2xl font-bold mb-4">Patients</h2>
                <p>Patients page</p>
              </div>
            }
          />
          <Route
            path="/doctors"
            element={
              <div>
                <h2 className="text-2xl font-bold mb-4">Doctors</h2>
                <p>Doctors page</p>
              </div>
            }
          />
        </Routes>
      </main>
      <Footer />
      <ToastContainer />
    </HashRouter>
  );
}

export default App;
=== END ===

=== FILE: src/main.jsx ===
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
=== END ===

=== FILE: src/index.css ===
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  @apply bg-gray-100;
}
=== END ===

=== FILE: src/api.js ===
// No code needed in this file as API calls are made directly in the components
=== END ===