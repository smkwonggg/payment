from flask import Flask, render_template, request, redirect, url_for
import pandas as pd
import numpy as np

app = Flask(__name__)
@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        print(request.form)
        
        names_payment = {}
        names_payment_responsible = {}
        
        # Process each payment entry
        for name, payment, responsible in zip(request.form.getlist('name'), 
                                              request.form.getlist('payment'), 
                                              request.form.getlist('responsible')):
            if name and payment:
                names_payment[name] = float(payment)
                responsible_person = responsible.split(',')
                names_payment_responsible[name] = {'payment': float(payment), 'responsible person': responsible_person}

        # Calculate amounts payable
        names_payable = {}
        for name_info in names_payment_responsible.values():
            payment = name_info['payment']
            responsible_person = name_info['responsible person']
            shared_amount = payment / len(responsible_person)
            for name in responsible_person:
                if name not in names_payable:
                    names_payable[name] = 0
                names_payable[name] += shared_amount

        # Calculate residuals
        keys = list(names_payment.keys())
        result_array = np.array(list(names_payment.values())) - np.array(list(names_payable.values())) 
        names_residual = dict(zip(keys, result_array))
        names_residual = {k: float(v) for k, v in names_residual.items()}

        debtors = {}
        creditors = {}
        for name, residual in names_residual.items():
            if residual < 0:
                debtors[name] = abs(residual)
            else:
                creditors[name] = abs(residual)

        debtors_rev_sorted = dict(sorted(debtors.items(), key=lambda debtors: debtors[1], reverse=True))
        creditors_rev_sorted = dict(sorted(creditors.items(), key=lambda creditors: creditors[1], reverse=True))

        debtors_name_list = list(debtors_rev_sorted.keys())
        creditors_name_list = list(creditors_rev_sorted.keys())
        debtors_residual_list = list(debtors_rev_sorted.values())
        creditors_residual_list = list(creditors_rev_sorted.values())

        # Process transactions
        transactions = []
        dr_no = 0
        cr_no = 0
        while dr_no < len(debtors_residual_list) and cr_no < len(creditors_residual_list):
            debtor_name = debtors_name_list[dr_no]
            creditor_name = creditors_name_list[cr_no]
            debtor = debtors_residual_list[dr_no]
            creditor = creditors_residual_list[cr_no]
            if creditor >= debtor:
                transactions.append(f"{debtor_name} should pay {creditor_name} ${debtor:.2f}")
                creditors_residual_list[cr_no] -= debtor
                debtors_residual_list[dr_no] = 0
                dr_no += 1
            else:
                transactions.append(f"{debtor_name} should pay {creditor_name} ${creditor:.2f}")
                debtors_residual_list[dr_no] -= creditor
                creditors_residual_list[cr_no] = 0
                cr_no += 1
        print("Transactions:", transactions)  # Add this line before rendering
        return render_template('result.html', transactions=transactions)
    return render_template('index.html')  # Change to your HTML file name

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
