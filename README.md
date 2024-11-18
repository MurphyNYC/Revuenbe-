# Revuenbe-
import React, { useState, useEffect } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Plus, Trash2, Save, Download } from "lucide-react";

const EventAccountingTracker = () => {
  // Load saved data on component mount
  const loadSavedData = () => {
    try {
      const savedData = localStorage.getItem('eventAccountingData');
      if (savedData) {
        const parsed = JSON.parse(savedData);
        setEventDetails(parsed.eventDetails);
        setRevenue(parsed.revenue);
        setExpenses(parsed.expenses);
        setPayouts(parsed.payouts);
        setAdditionalNotes(parsed.additionalNotes);
      }
    } catch (error) {
      console.error('Error loading saved data:', error);
    }
  };

  // Initialize state with empty values
  const [eventDetails, setEventDetails] = useState({
    name: '',
    venue: '',
    date: '',
    notes: ''
  });

  const [revenue, setRevenue] = useState([
    { source: '', amount: '', notes: '' }
  ]);

  const [expenses, setExpenses] = useState([
    { type: '', amount: '', vendor: '' }
  ]);

  const [payouts, setPayouts] = useState([
    { recipient: '', amount: '', percentage: '' }
  ]);

  const [additionalNotes, setAdditionalNotes] = useState('');

  // Load data on component mount
  useEffect(() => {
    loadSavedData();
  }, []);

  const saveData = () => {
    try {
      const dataToSave = {
        eventDetails,
        revenue,
        expenses,
        payouts,
        additionalNotes
      };
      localStorage.setItem('eventAccountingData', JSON.stringify(dataToSave));
      alert('Data saved successfully!');
    } catch (error) {
      console.error('Error saving data:', error);
      alert('Error saving data. Please try again.');
    }
  };

  const exportData = () => {
    try {
      const dataToExport = {
        eventDetails,
        revenue,
        expenses,
        payouts,
        additionalNotes,
        summary: {
          totalRevenue: calculateTotal(revenue),
          totalExpenses: calculateTotal(expenses),
          totalPayouts: calculateTotal(payouts),
          netProfit: calculateTotal(revenue) - calculateTotal(expenses)
        }
      };
      
      const blob = new Blob([JSON.stringify(dataToExport, null, 2)], { type: 'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `event-accounting-${eventDetails.name || 'untitled'}.json`;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
    } catch (error) {
      console.error('Error exporting data:', error);
      alert('Error exporting data. Please try again.');
    }
  };

  // Rest of the original functions
  const calculateTotal = (items, field = 'amount') => {
    return items.reduce((sum, item) => {
      const value = parseFloat(item[field]) || 0;
      return sum + value;
    }, 0);
  };

  const handleAddRow = (type, setter) => {
    if (type === 'revenue') {
      setter([...revenue, { source: '', amount: '', notes: '' }]);
    } else if (type === 'expenses') {
      setter([...expenses, { type: '', amount: '', vendor: '' }]);
    } else if (type === 'payouts') {
      setter([...payouts, { recipient: '', amount: '', percentage: '' }]);
    }
  };

  const handleRemoveRow = (type, index, setter, items) => {
    const newItems = items.filter((_, i) => i !== index);
    setter(newItems);
  };

  const handleInputChange = (type, index, field, value, setter, items) => {
    const newItems = [...items];
    newItems[index] = { ...newItems[index], [field]: value };
    setter(newItems);
  };

  // Add save and export buttons at the top
  const ActionButtons = () => (
    <div className="flex justify-end gap-4 mb-6">
      <button
        onClick={saveData}
        className="flex items-center gap-2 bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700"
      >
        <Save size={20} /> Save Event Data
      </button>
      <button
        onClick={exportData}
        className="flex items-center gap-2 bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700"
      >
        <Download size={20} /> Export Data
      </button>
    </div>
  );

  // The original JSX return with ActionButtons added
  return (
    <div className="max-w-4xl mx-auto p-4 space-y-6">
      <ActionButtons />
      <Card>
        <CardHeader>
          <CardTitle>Event Accounting Tracker</CardTitle>
        </CardHeader>
        <CardContent className="space-y-6">
          {/* Event Details Section */}
          <div className="space-y-4">
            <h3 className="text-lg font-semibold">Event Details</h3>
            <div className="grid grid-cols-2 gap-4">
              <input
                type="text"
                placeholder="Event Name"
                className="border p-2 rounded"
                value={eventDetails.name}
                onChange={(e) => setEventDetails({...eventDetails, name: e.target.value})}
              />
              <input
                type="text"
                placeholder="Venue"
                className="border p-2 rounded"
                value={eventDetails.venue}
                onChange={(e) => setEventDetails({...eventDetails, venue: e.target.value})}
              />
              <input
                type="date"
                className="border p-2 rounded"
                value={eventDetails.date}
                onChange={(e) => setEventDetails({...eventDetails, date: e.target.value})}
              />
              <input
                type="text"
                placeholder="Notes"
                className="border p-2 rounded"
                value={eventDetails.notes}
                onChange={(e) => setEventDetails({...eventDetails, notes: e.target.value})}
              />
            </div>
          </div>

          {/* Rest of the original components */}
          {/* Revenue Section */}
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <h3 className="text-lg font-semibold">Revenue</h3>
              <button
                onClick={() => handleAddRow('revenue', setRevenue)}
                className="flex items-center text-blue-600 hover:text-blue-800"
              >
                <Plus size={20} /> Add Row
              </button>
            </div>
            {revenue.map((item, index) => (
              <div key={index} className="flex gap-4 items-center">
                <input
                  type="text"
                  placeholder="Source"
                  className="border p-2 rounded flex-1"
                  value={item.source}
                  onChange={(e) => handleInputChange('revenue', index, 'source', e.target.value, setRevenue, revenue)}
                />
                <input
                  type="number"
                  placeholder="Amount"
                  className="border p-2 rounded w-32"
                  value={item.amount}
                  onChange={(e) => handleInputChange('revenue', index, 'amount', e.target.value, setRevenue, revenue)}
                />
                <input
                  type="text"
                  placeholder="Notes"
                  className="border p-2 rounded flex-1"
                  value={item.notes}
                  onChange={(e) => handleInputChange('revenue', index, 'notes', e.target.value, setRevenue, revenue)}
                />
                <button
                  onClick={() => handleRemoveRow('revenue', index, setRevenue, revenue)}
                  className="text-red-600 hover:text-red-800"
                >
                  <Trash2 size={20} />
                </button>
              </div>
            ))}
            <div className="text-right font-semibold">
              Total Revenue: ${calculateTotal(revenue).toFixed(2)}
            </div>
          </div>

          {/* Expenses Section */}
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <h3 className="text-lg font-semibold">Expenses</h3>
              <button
                onClick={() => handleAddRow('expenses', setExpenses)}
                className="flex items-center text-blue-600 hover:text-blue-800"
              >
                <Plus size={20} /> Add Row
              </button>
            </div>
            {expenses.map((item, index) => (
              <div key={index} className="flex gap-4 items-center">
                <input
                  type="text"
                  placeholder="Type"
                  className="border p-2 rounded flex-1"
                  value={item.type}
                  onChange={(e) => handleInputChange('expenses', index, 'type', e.target.value, setExpenses, expenses)}
                />
                <input
                  type="number"
                  placeholder="Amount"
                  className="border p-2 rounded w-32"
                  value={item.amount}
                  onChange={(e) => handleInputChange('expenses', index, 'amount', e.target.value, setExpenses, expenses)}
                />
                <input
                  type="text"
                  placeholder="Vendor"
                  className="border p-2 rounded flex-1"
                  value={item.vendor}
                  onChange={(e) => handleInputChange('expenses', index, 'vendor', e.target.value, setExpenses, expenses)}
                />
                <button
                  onClick={() => handleRemoveRow('expenses', index, setExpenses, expenses)}
                  className="text-red-600 hover:text-red-800"
                >
                  <Trash2 size={20} />
                </button>
              </div>
            ))}
            <div className="text-right font-semibold">
              Total Expenses: ${calculateTotal(expenses).toFixed(2)}
            </div>
          </div>

          {/* Net Profit Section */}
          <div className="bg-gray-100 p-4 rounded-lg">
            <h3 className="text-lg font-semibold mb-2">Net Profit Calculation</h3>
            <div className="space-y-2">
              <div className="flex justify-between">
                <span>Total Revenue:</span>
                <span>${calculateTotal(revenue).toFixed(2)}</span>
              </div>
              <div className="flex justify-between">
                <span>Total Expenses:</span>
                <span>${calculateTotal(expenses).toFixed(2)}</span>
              </div>
              <div className="flex justify-between font-bold">
                <span>Net Profit:</span>
                <span>${(calculateTotal(revenue) - calculateTotal(expenses)).toFixed(2)}</span>
              </div>
            </div>
          </div>

          {/* Payouts Section */}
          <div className="space-y-4">
            <div className="flex justify-between items-center">
              <h3 className="text-lg font-semibold">Payouts</h3>
              <button
                onClick={() => handleAddRow('payouts', setPayouts)}
                className="flex items-center text-blue-600 hover:text-blue-800"
              >
                <Plus size={20} /> Add Row
              </button>
            </div>
            {payouts.map((item, index) => (
              <div key={index} className="flex gap-4 items-center">
                <input
                  type="text"
                  placeholder="Recipient"
                  className="border p-2 rounded flex-1"
                  value={item.recipient}
                  onChange={(e) => handleInputChange('payouts', index, 'recipient', e.target.value, setPayouts, payouts)}
                />
                <input
                  type="number"
                  placeholder="Amount"
                  className="border p-2 rounded w-32"
                  value={item.amount}
                  onChange={(e) => handleInputChange('payouts', index, 'amount', e.target.value, setPayouts, payouts)}
                />
                <input
                  type="number"
                  placeholder="Percentage"
                  className="border p-2 rounded w-32"
                  value={item.percentage}
                  onChange={(e) => handleInputChange('payouts', index, 'percentage', e.target.value, setPayouts, payouts)}
                />
                <button
                  onClick={() => handleRemoveRow('payouts', index, setPayouts, payouts)}
                  className="text-red-600 hover:text-red-800"
                >
                  <Trash2 size={20} />
                </button>
              </div>
            ))}
            <div className="text-right font-semibold">
              Total Payouts: ${calculateTotal(payouts).toFixed(2)}
            </div>
          </div>

          {/* Additional Notes Section */}
          <div className="space-y-2">
            <h3 className="text-lg font-semibold">Additional Notes</h3>
            <textarea
              className="w-full border p-2 rounded h-24"
              placeholder="Additional observations or comments..."
              value={additionalNotes}
              onChange={(e) => setAdditionalNotes(e.target.value)}
            />
          </div>
        </CardContent>
      </Card>
    </div>
  );
};

export default EventAccountingTracker;
