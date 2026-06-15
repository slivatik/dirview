#!/usr/bin/env python3
# -*- coding: utf-8 -*-

import sys
import os
from pathlib import Path
from datetime import datetime

from PyQt5.QtWidgets import (
    QApplication, QMainWindow, QTreeWidget, QTreeWidgetItem,
    QWidget, QVBoxLayout, QLineEdit, QPushButton, QHBoxLayout, QLabel
)
from PyQt5.QtCore import Qt
from PyQt5.QtGui import QIcon


class DirView(QMainWindow):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("DirView - Файловый менеджер")
        self.resize(1100, 700)

        # Поле фильтрации (ТЗ)
        self.filter_input = QLineEdit()
        self.filter_input.setPlaceholderText("Фильтр по имени...")
        self.filter_input.textChanged.connect(self.filter_tree)

        # Дерево
        self.tree = QTreeWidget()
        self.tree.setHeaderLabels(["Имя", "Размер", "Тип", "Дата изменения", "Размер папки"])
        self.tree.setColumnWidth(0, 350)
        self.tree.setColumnWidth(1, 80)
        self.tree.setColumnWidth(2, 100)
        self.tree.setColumnWidth(3, 150)
        self.tree.setColumnWidth(4, 250)

        # Включаем сортировку
        self.tree.setSortingEnabled(True)

        # Загружаем домашнюю папку (ТЗ)
        home_dir = os.path.expanduser("~")
        self.load_folder(home_dir, None)

        # Двойной клик для открытия папок
        self.tree.itemDoubleClicked.connect(self.on_double_click)

        # Layout
        layout = QVBoxLayout()
        layout.addWidget(self.filter_input)
        layout.addWidget(self.tree)

        central = QWidget()
        central.setLayout(layout)
        self.setCentralWidget(central)

    def get_icon(self, path):
        """Иконка для папок"""
        if os.path.isdir(path):
            return QIcon.fromTheme("folder", QIcon())
        return QIcon.fromTheme("text-x-generic", QIcon())

    def load_folder(self, path, parent_item):
        """Загружает содержимое папки (не рекурсивно)"""
        try:
            items = os.listdir(path)
        except PermissionError:
            return

        folders = []
        files = []
        for name in items:
            full = os.path.join(path, name)
            if os.path.isdir(full):
                folders.append(name)
            else:
                files.append(name)

        folders.sort(key=str.lower)
        files.sort(key=str.lower)

        # Папки
        for name in folders:
            full_path = os.path.join(path, name)
            self.add_item(full_path, name, parent_item, is_folder=True)

        # Файлы
        for name in files:
            full_path = os.path.join(path, name)
            self.add_item(full_path, name, parent_item, is_folder=False)

    def add_item(self, full_path, name, parent_item, is_folder):
        """Создаёт один элемент дерева"""
        if parent_item is None:
            item = QTreeWidgetItem(self.tree)
        else:
            item = QTreeWidgetItem(parent_item)

        item.setText(0, name)
        item.setIcon(0, self.get_icon(full_path))
        item.setData(0, Qt.UserRole, full_path)

        if is_folder:
            item.setText(2, "Папка")
            item.addChild(QTreeWidgetItem())  # Для стрелки раскрытия

            # Виджет с кнопкой и меткой
            container = QWidget()
            layout = QHBoxLayout()
            layout.setContentsMargins(0, 0, 0, 0)

            btn = QPushButton("Обновить")
            btn.setFixedWidth(80)
            label = QLabel("")
            label.setVisible(False)

            btn.clicked.connect(lambda checked, p=full_path, b=btn, l=label: self.calc_size(p, b, l))

            layout.addWidget(btn)
            layout.addWidget(label)
            layout.addStretch()
            container.setLayout(layout)

            self.tree.setItemWidget(item, 4, container)

            item.btn = btn
            item.size_label = label
            item.loaded = False
        else:
            # Файл
            try:
                size = os.path.getsize(full_path)
                item.setText(1, self.format_size(size))
                item.setData(1, Qt.UserRole, size)
            except:
                item.setText(1, "?")

            ext = os.path.splitext(name)[1].upper()
            item.setText(2, ext + " файл" if ext else "Файл")

        # Дата изменения
        try:
            mtime = os.path.getmtime(full_path)
            date_str = datetime.fromtimestamp(mtime).strftime("%d.%m.%Y %H:%M")
            item.setText(3, date_str)
            item.setData(3, Qt.UserRole, mtime)
        except:
            pass

    def calc_size(self, path, button, label):
        """Расчёт размера папки"""
        button.setEnabled(False)
        button.setText("Расчёт...")
        QApplication.processEvents()

        total = 0
        try:
            for f in Path(path).rglob("*"):
                if f.is_file():
                    try:
                        total += f.stat().st_size
                    except:
                        pass
            button.setText("Обновить")
            button.setEnabled(True)
            label.setText(self.format_size(total))
            label.setVisible(True)
        except:
            button.setText("Обновить")
            button.setEnabled(True)
            label.setText("Ошибка")
            label.setVisible(True)

    def format_size(self, size):
        for unit in ['Б', 'КБ', 'МБ', 'ГБ']:
            if size < 1024:
                return f"{size:.1f} {unit}"
            size /= 1024
        return f"{size:.1f} ТБ"

    def filter_tree(self, text):
        """Рекурсивная фильтрация по имени"""
        text = text.lower()
        root = self.tree.invisibleRootItem()
        self.filter_items(root, text)

    def filter_items(self, parent, text):
        for i in range(parent.childCount()):
            item = parent.child(i)
            visible = text == "" or text in item.text(0).lower()
            item.setHidden(not visible)
            if visible:
                self.filter_items(item, text)

    def on_double_click(self, item, col):
        path = item.data(0, Qt.UserRole)
        if os.path.isdir(path):
            if item.isExpanded():
                item.setExpanded(False)
            else:
                if not getattr(item, 'loaded', False):
                    if item.childCount() > 0:
                        temp = item.child(0)
                        if temp.text(0) == "":
                            item.removeChild(temp)
                    self.load_folder(path, item)
                    item.loaded = True
                item.setExpanded(True)


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = DirView()
    window.show()
    sys.exit(app.exec_())
