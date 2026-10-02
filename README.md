<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Bike Booking Manager</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    body { font-family: Arial; padding: 15px; background: #f5f5f5; margin: 0; }
    h1 { color: #333; font-size: 22px; }
    input, select, button { 
      width: 100%; padding: 10px; margin: 5px 0; 
      border: 1px solid #ccc; border-radius: 5px; font-size: 16px;
      box-sizing: border-box;
    }
    button { background: #2563eb; color: white; border: none; cursor: pointer; }
    button:active { background: #1e40af; }
    .card {
      background: white; padding: 12px; margin: 10px 0;
      border-radius: 8px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);
    }
    .status { padding: 3px 8px; border-radius: 4px; font-size: 12px; }
    .Pending { background: #e5e7eb; }
    .InProgress { background: #fef3c7; }
    .Ready { background: #dbeafe; }
    .Delivered { background: #d1fae5; }
    .btn-sm { width: auto; padding: 5px 10px; font-size: 13px; margin: 2px; display: inline-block; }
    .delete { background: #dc2626; }
  </style>
</head>
<body>
  <h1>🏍️ Bike Booking Manager</h1>

  <div class="card">
    <input id="name" placeholder="Customer Name" />
    <input id="mobile" placeholder="Mobile Number" type="tel" />
    <input id="bike" placeholder="Bike Model" />
    <select id="status">
      <option>Pending</option>
      <option>In Progress</option>
      <option>Ready</option>
      <option>Delivered</option>
    </select>
    <input id="date" type="date" />
    <input id="remarks" placeholder="Remarks" />
    <button onclick="saveBooking()">Add Booking</button>
  </div>

  <input id="search" placeholder="🔍 Search..." oninput="renderList()" style="margin-top:15px;" />

  <div id="list"></div>

  <script>
    // 🔴 MĪ SUPABASE VALUES IKKADA PETTANDI
    const SUPABASE_URL = 'మీ_Project_URL_ఇక్కడ';
    const SUPABASE_KEY = 'మీ_anon_key_ఇక్కడ';

    const supabase = window.supabase.createClient(SUPABASE_URL, SUPABASE_KEY);
    let bookings = [];
    let editId = null;

    async function loadBookings() {
      const { data, error } = await supabase
        .from('bookings').select('*')
        .order('created_at', { ascending: false });
      if (error) { alert('Error: ' + error.message); return; }
      bookings = data || [];
      renderList();
    }

    async function saveBooking() {
      const form = {
        customer_name: document.getElementById('name').value,
        mobile_number: document.getElementById('mobile').value,
        bike_model: document.getElementById('bike').value,
        booking_status: document.getElementById('status').value,
        delivery_date: document.getElementById('date').value || null,
        remarks: document.getElementById('remarks').value
      };

      if (!form.customer_name || !form.mobile_number || !form.bike_model) {
        alert('Name, Mobile, Bike fill cheyandi');
        return;
      }

      let error;
      if (editId) {
        ({ error } = await supabase.from('bookings').update(form).eq('id', editId));
        editId = null;
      } else {
        ({ error } = await supabase.from('bookings').insert([form]));
      }

      if (error) { alert('Error: ' + error.message); return; }

      document.getElementById('name').value = '';
      document.getElementById('mobile').value = '';
      document.getElementById('bike').value = '';
      document.getElementById('date').value = '';
      document.getElementById('remarks').value = '';
      loadBookings();
    }

    function editBooking(id) {
      const b = bookings.find(x => x.id === id);
      document.getElementById('name').value = b.customer_name;
      document.getElementById('mobile').value = b.mobile_number;
      document.getElementById('bike').value = b.bike_model;
      document.getElementById('status').value = b.booking_status;
      document.getElementById('date').value = b.delivery_date || '';
      document.getElementById('remarks').value = b.remarks || '';
      editId = id;
      window.scrollTo(0, 0);
    }

    async function deleteBooking(id) {
      if (!confirm('Delete cheyyala?')) return;
      const { error } = await supabase.from('bookings').delete().eq('id', id);
      if (error) { alert('Error: ' + error.message); return; }
      loadBookings();
    }

    function renderList() {
      const search = document.getElementById('search').value.toLowerCase();
      const filtered = bookings.filter(b =>
        b.customer_name?.toLowerCase().includes(search) ||
        b.mobile_number?.includes(search) ||
        b.bike_model?.toLowerCase().includes(search)
      );

      document.getElementById('list').innerHTML = filtered.map(b => `
        <div class="card">
          <b>${b.customer_name}</b><br/>
          📱 ${b.mobile_number}<br/>
          🏍️ ${b.bike_model}<br/>
          <span class="status ${b.booking_status.replace(' ', '')}">${b.booking_status}</span>
          ${b.delivery_date ? '📅 ' + b.delivery_date : ''}<br/>
          ${b.remarks ? '📝 ' + b.remarks : ''}<br/>
          <button class="btn-sm" onclick="editBooking(${b.id})">Edit</button>
          <button class="btn-sm delete" onclick="deleteBooking(${b.id})">Delete</button>
        </div>
      `).join('');
    }

    loadBookings();
  </script>
</body>
</html>
